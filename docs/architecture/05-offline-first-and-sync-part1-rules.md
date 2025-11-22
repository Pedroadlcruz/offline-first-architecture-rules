---
trigger: glob
globs: lib/**/*.dart
---

Offline-First & Sync Rules
1. Offline-First Strategy
The project follows an offline-first strategy where:


The local database is the Single Source of Truth (SSoT).


All reads go to the local DB first.


Writes are stored locally and enqueued in an outbox for later sync.


Synchronization with the server happens in background.


The UI always displays local data, guaranteeing immediate feedback.



2. Local Database as SSoT
2.1 Principles


Always read from local first
Every data query must read from the local database or cache.


Background sync
When network is available, sync in the background without blocking the UI.


Optimistic operations
Writes are confirmed locally immediately. The user sees the update without waiting for the server.


Outbox for pending operations
Any operation that must be reflected on the server is added to an outbox.


2.2 Read Flow
User requests data
      ↓
Read from local DB
      ↓
Return local data immediately
      ↓
[Background] If network is available → trigger sync with server
      ↓
[Background] Update local DB with remote changes
      ↓
UI automatically updates from local DB (streams / observers)


3. Outbox Pattern
3.1 Concept
The outbox is a local store (table/collection) that tracks pending write operations that must be sent to the server.
Each outbox entry represents a command such as “create this entity on the server”, “update this entity”, or “delete this entity”.
3.2 Outbox Structure (Generic)
Fields typically include:


id: unique identifier for the outbox entry (UUID).


entityType: entity name or type (e.g. "Planning", "Visit").


localEntityId: ID of the related local entity.


action: "create" | "update" | "delete".


payloadJson: serialized snapshot of the request payload.


createdAt: when the operation was enqueued.


sentAt: when the operation was successfully sent (nullable).


status: "pending" | "sent" | "error".


lastErrorMessage: last error, if any (nullable).


lastErrorCode: last error code, if any (nullable).


Example (pseudo-code):
class OutboxEntry {
  final String id;
  final String entityType;
  final String localEntityId;
  final String action; // "create" | "update" | "delete"
  final String payloadJson;
  final DateTime createdAt;
  final DateTime? sentAt;
  final String status; // "pending" | "sent" | "error"
  final String? lastErrorMessage;
  final String? lastErrorCode;

  // ...
}

3.3 Operation Types


create – create a new entity on the server.


update – update an existing entity on the server.


delete – delete or soft-delete an entity on the server.


3.4 Enqueueing Operations
Whenever the app performs a write that must be synced:


Write to local DB (insert/update/delete).


Create an outbox entry with the appropriate payload.


Example (pseudo-code):
await outboxLocalDataSource.enqueue(
  entityType: 'Planning',
  localEntityId: planningId,
  action: OutboxAction.create,
  payloadJson: jsonEncode(payload),
);

3.5 Sending & Confirming Operations
Background process (e.g. worker, scheduled task):
final pendingOps = await outboxLocalDataSource.getPending(
  entityType: 'Planning',
  limit: 20,
);

for (final op in pendingOps) {
  final result = await outboxRemoteDataSource.sendOperation(op);

  if (result.success && result.remoteId != null) {
    // Update local entity with remote metadata (id, version, timestamps, etc.)
    await localDataSource.applyServerResult(
      localId: op.localEntityId,
      remoteId: result.remoteId!,
      newVersion: result.newVersion,
      serverTimestampUtc: result.serverTimestampUtc,
    );

    // Mark outbox entry as sent
    await outboxLocalDataSource.markAsSent(op.id);
  } else {
    await outboxLocalDataSource.markAsError(
      outboxId: op.id,
      errorMessage: result.errorMessage ?? 'Unknown error',
      errorCode: result.errorCode,
    );
  }
}

3.6 Retry Handling
Errors should be retryable:
await outboxLocalDataSource.requeue(outboxId);
// status: "error" → "pending"

Retries should use backoff (fixed, exponential, jitter, etc.), configured in the sync layer.

4. Delta Synchronization
4.1 Concept
Delta sync means downloading only changes since the last sync, not the entire dataset.
Typical approaches:


Timestamp (lastSync).


Version number or sequence.


Server-side cursor or token.


4.2 Using lastSync Timestamp
Generic flow:
// 1. Read last sync metadata
final lastSync = await syncLocalDataSource.getLastSyncDate();

// 2. Ask server for changes since lastSync
final remoteChanges = await syncRemoteDataSource.fetchModifiedSince(lastSync);

// 3. Apply changes to local DB
await syncLocalDataSource.saveSyncData(remoteChanges);

// 4. Update lastSync
await syncLocalDataSource.updateLastSyncDate(DateTime.now().toUtc());

4.3 Per-Table / Per-Entity Sync


Each table/entity type can have its own sync configuration.


lastSync can be:


Global (one timestamp for everything), or


Per entity type (recommended for large systems).




4.4 Conflict Handling
Common strategies:


Last Write Wins
The last successful write (by timestamp or version) wins.


Server Wins
Server state overwrites client state in conflicts.


Client Wins
Client state overwrites server state.


Manual Resolution
Conflicting data is flagged, and the user (or a backoffice tool) resolves it.


Example (pseudo-code for version conflict):
if (result.errorCode == 'VERSION_CONFLICT') {
  // Option A: Pull remote changes first, then requeue local op
  await syncRepository.pullRemoteChanges();
  await outboxLocalDataSource.requeue(op.id);

  // Option B: Mark as error for manual resolution
  await outboxLocalDataSource.markAsError(
    outboxId: op.id,
    errorMessage: 'Version conflict',
    errorCode: 'VERSION_CONFLICT',
  );
}


5. Responsibilities by Layer
5.1 Data Layer
Outbox Local Data Source
Responsibilities:


Enqueue operations (enqueue).


Fetch pending operations (getPending).


Mark operations as sent (markAsSent).


Mark operations as error (markAsError).


Requeue operations (requeue).


Count pending/error items.


Outbox Remote Data Source
Responsibilities:


Send outbox operations to the server (sendOperation).


Parse server responses (remote IDs, versions, timestamps).


Map network errors to data-layer errors.


Sync Local Data Source
Responsibilities:


Store and update sync metadata (e.g. lastSync).


Apply remote changes to local DB (saveSyncData).


Handle per-table/per-entity sync configuration.


Sync Remote Data Source
Responsibilities:


Fetch remote changes since last sync (fetchModifiedSince).


Interact with dedicated sync endpoints (delta APIs).


Handle pagination/cursors where applicable.



5.2 Domain Layer
Outbox Repository (Interface)
Responsibilities:


Define the contract for outbox operations.


Abstract underlying local/remote data sources.


Sync Repository (Interface)
Responsibilities:


Define high-level sync operations:


pushLocalChanges()


pullRemoteChanges()


syncBaseData() / syncEntityType(...)


getLastSyncDate()


getPendingSyncCount()




Hide implementation details from the presentation layer.