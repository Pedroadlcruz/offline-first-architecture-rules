---
trigger: glob
globs: lib/**/domain/**/*.dart
---

# Domain Layer Rules

## 1. Purpose of the Domain Layer

The **domain layer** is the core of the application. It contains:

* **Entities** – pure business objects (no framework dependencies).
* **Value Objects** – immutable objects that represent concepts such as amounts, coordinates, ranges, etc.
* **Repository Interfaces** – contracts for data access (no implementations).
* **Use Cases / Interactors** – application-specific business logic that coordinates multiple repositories and applies rules.

The domain layer should be **independent** and must not depend on other layers (data, presentation, infrastructure, UI frameworks).

---

## 2. Allowed Responsibilities

### 2.1 Entities

Entities:

* Represent **business concepts** (User, Order, Visit, etc.).
* Contain **properties** and **simple business methods**.
* Are **immutable** when possible (mutations via `copyWith` or equivalent).
* Can enforce **basic domain invariants** (e.g. non-empty IDs, simple constraints).

Example (simplified):

```dart
class Customer {
  final String id;
  final String name;
  final String email;

  const Customer({
    required this.id,
    required this.name,
    required this.email,
  });

  bool get hasValidEmail => email.contains('@');

  Customer copyWith({
    String? id,
    String? name,
    String? email,
  }) {
    return Customer(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
    );
  }
}
```

---

### 2.2 Repository Interfaces

Repository interfaces:

* Define **contracts** for data access and persistence.
* Do **not** contain implementation details.
* Work with **domain entities and value objects**, never data-layer models.
* Return a **result abstraction** (`Result<T>`, `Either<Failure, T>`, `Stream<T>`, etc.).

Example:

```dart
abstract class CustomerRepository {
  Future<Result<List<Customer>>> getAllCustomers();
  Future<Result<Customer?>> getCustomerById(String id);
  Future<Result<void>> saveCustomer(Customer customer);
  Stream<List<Customer>> watchCustomers();
}
```

---

### 2.3 Use Cases / Interactors

Use cases:

* Encapsulate **application-specific business logic**.
* Coordinate **multiple repositories** and value objects.
* Apply **business rules, validations, and calculations**.
* Expose a **single public entry point** (e.g. `call()` or `execute()`).
* Can define input parameter objects (`Params`) and return domain types or `Result<T>`.

Example:

```dart
class CreateOrder {
  final CustomerRepository _customerRepository;
  final OrderRepository _orderRepository;

  CreateOrder({
    required CustomerRepository customerRepository,
    required OrderRepository orderRepository,
  })  : _customerRepository = customerRepository,
        _orderRepository = orderRepository;

  Future<Result<String>> call(CreateOrderParams params) async {
    // 1. Validate business rules.
    // 2. Fetch required domain data.
    // 3. Build entity/entities.
    // 4. Persist through repositories.
    // 5. Return Result with new ID or failure.
  }
}
```

---

### 2.4 Value Objects

Value objects:

* Represent **non-entity concepts** (Money, Email, TimeRange, FormContext, etc.).
* Are **immutable**.
* Enforce their **own invariants and validation**.
* Implement equality based on **value**, not identity.

Example:

```dart
class Money {
  final double amount;
  final String currency;

  const Money(this.amount, this.currency)
      : assert(amount >= 0),
        assert(currency.length == 3);

  bool get isZero => amount == 0;

  Money add(Money other) {
    if (other.currency != currency) {
      throw ArgumentError('Currency mismatch');
    }
    return Money(amount + other.amount, currency);
  }
}
```

---

## 3. Forbidden Responsibilities

The domain layer **must not**:

* Contain **concrete implementations** of repositories, data sources, or services.
* Depend on **frameworks** (no Flutter widgets, HTTP, DB, UI libraries, etc.).
* Access databases, network clients, file systems, or any infrastructure APIs.
* Hold **UI logic** (state management, widgets, controllers, etc.).
* Perform persistence or JSON (de)serialization (except minimal support in value objects when strictly necessary and still framework-independent).

---

## 4. Dependency Rules

### 4.1 Allowed Dependencies

The domain layer may depend on:

* **Domain types**:

  * Entities.
  * Value objects.
  * Enums and simple domain types.
* **Shared abstractions**:

  * `Result`, `Failure` / `AppFailure`.
  * Simple utilities like `UuidGenerator` or `Clock` abstractions (no concrete implementations).

### 4.2 Forbidden Dependencies

The domain layer must not depend on:

* `data/` layer (models, repositories implementations, data sources).
* `presentation/` layer (UI, controllers, BLoCs, view models).
* Platform or framework SDKs (Flutter, Android, iOS, HTTP libraries, database libraries, etc.).
* Infrastructure services (network clients, DAOs, API clients).

Allowed exception:

* The domain layer can expose **interfaces** for infrastructure (e.g. `TimeProvider`, `IdGenerator`) but **implementations** live elsewhere.

---

## 5. Use Case Creation Rules

### 5.1 When to Create a Use Case ✅

Create a use case when **any** of the following is true:

1. It **coordinates two or more repositories**
   (e.g. auth + user profile + orders).

2. It encapsulates **non-trivial business logic**, such as:

   * Complex validation rules.
   * Calculations or scoring.
   * Multi-step workflows.

3. It is reused by **multiple callers/features**
   (e.g. same operation is needed by different flows or screens).

Examples:

* Syncing data for a user across multiple aggregates.
* Creating an order that validates stock, customer credit, and pricing rules.
* Loading the “current user context” used by multiple parts of the app.

---

### 5.2 When NOT to Create a Use Case ❌

Avoid creating a use case when:

1. The operation simply **forwards** a call to a single repository without additional logic.

   ```dart
   // ❌ Not a real use case – just delegation
   class GetCustomerById {
     final CustomerRepository _repository;

     GetCustomerById(this._repository);

     Future<Result<Customer?>> call(String id) {
       return _repository.getCustomerById(id);
     }
   }
   ```

   In this case, prefer calling the **repository directly** from the caller (e.g. BLoC, controller).

2. The method is used **only in a single place** and there is no complex business logic.

3. It only **passes parameters through** without adding value (no additional rules, no transformations).

---

### 5.3 Decision Matrix for Use Cases

| Uses 2+ Repositories | Complex Business Logic | Used in Multiple Features | Should Be Use Case? |
| -------------------- | ---------------------- | ------------------------- | ------------------- |
| ✅ Yes                | ❌ No                   | ❌ No                      | ✅ YES               |
| ❌ No                 | ✅ Yes                  | ❌ No                      | ✅ YES               |
| ❌ No                 | ❌ No                   | ✅ Yes                     | ✅ YES               |
| ✅ Yes                | ✅ Yes                  | ❌ No                      | ✅ YES               |
| ✅ Yes                | ❌ No                   | ✅ Yes                     | ✅ YES               |
| ❌ No                 | ✅ Yes                  | ✅ Yes                     | ✅ YES               |
| ✅ Yes                | ✅ Yes                  | ✅ Yes                     | ✅ YES               |
| ❌ No                 | ❌ No                   | ❌ No                      | ❌ NO                |

---

## 6. Naming & Location Conventions

> Use “feature” or “module” as the generic term (e.g. `customers`, `orders`, `auth`).

### 6.1 Entities

* **File names**:
  Either `entity_name.dart` or `entity_name_entity.dart`
  e.g. `customer.dart`, `order.dart`
* **Location**:
  `features/{feature}/domain/entities/`
* **Convention**:
  Singular names; suffix `Entity` is optional but must be consistent project-wide.

### 6.2 Repositories (Interfaces)

* **File names**:
  `{feature}_repository.dart` or `{aggregate}_repository.dart`
* **Location**:
  `features/{feature}/domain/repositories/`
* **Note**:
  Only **interfaces** here; implementations live in the data layer and use `_impl` or other suffix.

### 6.3 Use Cases

* **File names**:
  `{action}_{target}_use_case.dart` or simply `{action}_{target}.dart`
  Examples: `get_customers.dart`, `create_order_use_case.dart`
* **Location**:
  `features/{feature}/domain/use_cases/` (or `usecases/`, but keep it consistent).
* **Naming**:
  Verb in present tense (Get, Create, Update, Sync, Validate, etc.).

### 6.4 Value Objects

* **File names**:
  `{concept}_value_object.dart` or just `{concept}.dart`
  Examples: `money.dart`, `email_address.dart`, `form_context.dart`
* **Location**:
  `features/{feature}/domain/value_objects/`
  or `core/domain/value_objects/` for cross-feature concepts.

---

## 7. Checklist for New Domain Code

Before merging a feature that touches the domain layer, verify:

* [ ] Entities are **immutable** or provide a clear immutable API (`copyWith`, etc.).
* [ ] Repository files in domain are **interfaces only** (no implementations, no infra details).
* [ ] Use cases are created **only when they add business value** (per rules above).
* [ ] No domain type imports **frameworks or infrastructure**.
* [ ] File names and locations follow the **naming conventions**.
* [ ] Entities and value objects **do not** contain UI logic.
* [ ] Value objects are immutable and enforce their own invariants.

---

*Domain layer architecture rules — generic, framework-agnostic, and reusable across projects.*
