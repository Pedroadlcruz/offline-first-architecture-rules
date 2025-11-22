---
trigger: glob
globs: lib/**/*.dart
---


# Offline-First & Sync Rules (Part 2)

## 5.3 Presentation Layer

**Sync Status State** (e.g. SyncStatusBloc / ViewModel)

Responsibilities:

* Expose:
  * Last sync date/time.
  * Number of pending items in the outbox.
  * Whether a sync is currently running.
* Trigger manual sync when requested by the user.

**Sync Widgets**

* `SyncStatusWidget` – displays high-level sync info (last sync, pending items).
* `SyncLoadingView` – used while a sync operation is in progress.
* `SyncSuccessView` – used after a successful sync (optional).

These should live in a shared UI module and be reused across screens.

## 6. End-to-End Sync Flow

### 6.1 User Action (Write)

```
User creates/updates/deletes an entity
      ↓
Write changes to local DB (immediate)
      ↓
Enqueue outbox operation (create/update/delete)
      ↓
UI updates immediately from local DB
```

### 6.2 Background: pushLocalChanges()

```
[Background] Check network connectivity
      ↓
[Background] Fetch pending outbox operations
      ↓
[Background] For each operation:
    - Send to server
    - On success:
        - Update local entity with remote metadata
        - Mark outbox entry as "sent"
    - On error:
        - Mark outbox entry as "error"
        - Optional: schedule retry with backoff
```

### 6.3 Background: pullRemoteChanges()

```
[Background] Check network connectivity
      ↓
[Background] Read lastSync metadata
      ↓
[Background] Fetch changes from server (delta since lastSync)
      ↓
[Background] Apply changes to local DB
      ↓
[Background] Update lastSync
      ↓
UI updates automatically from local DB (streams / observers)
```

### 6.4 UI Sync Status

Sync status state exposes:

* Last sync date/time
* Pending items count
* Sync in progress flag / error state
  ↓
Sync widgets show this information to the user

## 7. Naming Conventions

### 7.1 Outbox-Related Components

Use `outbox` in file/class names:

* `outbox_local_data_source.*`
* `outbox_remote_data_source.*`
* `outbox_repository.*`
* `outbox_entry.*`, `outbox_model.*`

### 7.2 Sync-Related Components

Use `sync` in file/class names:

* `sync_local_data_source.*`
* `sync_remote_data_source.*`
* `sync_repository.*`
* `sync_status_bloc.*` / `sync_status_view_model.*`

### 7.3 Database Tables / Collections

* `sync_outbox` or `outbox` for the outbox table/collection.
* Other sync metadata tables as needed
  (e.g. `sync_metadata`, `entity_sync_state`).

## 8. Checklist for New Offline-First Features

Before shipping a feature that participates in offline-first and sync, verify:

* [ ] Write operations save to local DB first (no direct remote-only writes).
* [ ] All write operations that must reach the server are enqueued in the outbox.
* [ ] All read operations use local DB as SSoT.
* [ ] Streams/observers are wired so UI updates from local changes.
* [ ] Sync logic uses delta sync or documents why it doesn't.
* [ ] Sync errors are logged and mapped to safe failures (no crashes).
* [ ] Version conflicts and edge cases have a documented strategy.
* [ ] Sync status (last sync, pending items) is visible in the UI where relevant.

## 9. Recommended Patterns (Short Examples)

### 9.1 Create Entity with Outbox (Repository Implementation)

```dart
Future<Result<String>> createPlanning(Planning planning) async {
  try {
    // 1. Save locally
    final localId = await planningLocalDataSource.insert(planning);

    // 2. Build payload for server
    final payload = PlanningMapper.toCreatePayload(planning.copyWith(id: localId));

    // 3. Enqueue in outbox
    await outboxLocalDataSource.enqueue(
      entityType: 'Planning',
      localEntityId: localId,
      action: OutboxAction.create,
      payloadJson: jsonEncode(payload),
    );

    return Result.success(localId);
  } catch (e) {
    return Result.failure(
      AppFailure.database(message: 'Failed to create planning: $e'),
    );
  }
}
```

### 9.2 Push Local Changes (Sync Repository Implementation)

```dart
Future<void> pushLocalChanges() async {
  if (!await networkInfo.isConnected) return;

  final ops = await outboxLocalDataSource.getPending(limit: 20);

  for (final op in ops) {
    try {
      final result = await outboxRemoteDataSource.sendOperation(op);

      if (result.success) {
        await localStore.applyServerResult(
          localId: op.localEntityId,
          remoteId: result.remoteId,
          newVersion: result.newVersion,
          serverTimestampUtc: result.serverTimestampUtc,
        );

        await outboxLocalDataSource.markAsSent(op.id);
      } else {
        await outboxLocalDataSource.markAsError(
          outboxId: op.id,
          errorMessage: result.errorMessage ?? 'Unknown error',
          errorCode: result.errorCode,
        );
      }
    } catch (e) {
      await outboxLocalDataSource.markAsError(
        outboxId: op.id,
        errorMessage: e.toString(),
      );
    }
  }
}
```

### 9.3 Pull Remote Changes (Sync Repository Implementation)

```dart
Future<void> pullRemoteChanges() async {
  if (!await networkInfo.isConnected) return;

  final lastSync = await syncLocalDataSource.getLastSyncDate();

  final changes = await syncRemoteDataSource.fetchModifiedSince(lastSync);

  await syncLocalDataSource.saveSyncData(changes);

  await syncLocalDataSource.updateLastSyncDate(DateTime.now().toUtc());
}
```

### 9.4 Sync Status in UI

```dart
class SyncStatusWidget extends StatelessWidget {
  const SyncStatusWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<SyncStatusBloc, SyncStatusState>(
      builder: (context, state) {
        return switch (state) {
          SyncStatusLoading() => const Text('Syncing...'),
          SyncStatusLoaded(:final lastSyncDate, :final pendingItems, :final isSyncing) => Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Last sync: ${_formatDate(lastSyncDate)}'),
                Text('Pending items: $pendingItems'),
                if (isSyncing) const Text('Sync in progress...'),
              ],
            ),
          SyncStatusError(:final message) => Text('Sync error: $message'),
          _ => const SizedBox.shrink(),
        };
      },
    );
  }

  String _formatDate(DateTime? date) {
    if (date == null) return 'Never';
    return '${date.toLocal()}';
  }
}
```

---

*Offline-first and synchronization architecture rules — generic, framework-agnostic, and reusable across projects.*