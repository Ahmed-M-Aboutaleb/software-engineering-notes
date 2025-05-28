# Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change. In other words, a class should have only one job or responsibility.

## The Problem

In the original example, the `User` class juggles three unrelated jobs:

1. **Modeling data** (storing `name` and `age`).
2. **Validating input** (`null` names and negative ages).
3. **Managing persistence** (adding itself to a global list).

Because these concerns can change for different reasons, the class violates SRP.

## Bad Example

```java
public class User {
    private String name;
    private int age;
    public static List<User> users = new ArrayList<User>();
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public void register() {
        if(this.name == null) {
            throw new NullPointerException("name is null");
        }
        if(this.age < 0) {
            throw new IllegalArgumentException("age is negative");
        }

        users.add(this);
    }
}
```

### Why This Violates SRP

- **Multiple reasons to change:**
    - Add or modify validation rules (e.g., require minimum age).
    - Change storage strategy (e.g., switch from in-memory list to database).
    - Extend the data model (e.g., add `email` or `address`).
- **Low cohesion:**
    - Data, validation logic, and persistence all live in one class.

## Refactored Solution

We’ll split responsibilities into four focused classes:

1. `User` - models immutable user data.
2. `UserRepository` - handles storage and retrieval.
3. `Validator` - checks input values.
4. `UserService` - orchestrates registration.

### 1. User (Data Model)

```java
public class User {
    private final String name;
    private final int age;
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() { return name; }
    public int getAge()    { return age; }
}
```

- **Responsibility:** Ensure any User instance is valid and immutable.

### 2. UserRepository (Persistence)

```java
public class UserRepository {
    private final List<User> users = new ArrayList<>();

    public void add(User user) {
        users.add(user);
    }

    public List<User> findAll() {
        return Collections.unmodifiableList(users);
    }
}
```

- **Responsibility:** Store and retrieve users.

### 3. Validator (Input Checking)

```java
public class Validator {
    public static void validate(String input) {
        if(input == null) {
            throw new NullPointerException("name is null");
        }
    }

    public static void validate(int input) {
        if(input < 0) {
            throw new IllegalArgumentException("age is negative");
        }
    }
}
```

- **Responsibility:** Centralize validation rules.

### 4. UserService (Business Logic)

```java
public class UserService {
    private final UserRepository repo;

    public UserService(UserRepository repo) {
        this.repo = repo;
    }
    public void register(User user) {
        Validator.validate(user.name);
        Validator.validate(user.age);
        this.repo.add(user);
    }
}
```

- **Responsibility:** Coordinate validation and persistence during registration.
