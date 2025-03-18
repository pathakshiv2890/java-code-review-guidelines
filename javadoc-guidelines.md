# Javadoc Guidelines

This document outlines the best practices and guidelines for writing Javadoc comments in your code. Following these guidelines ensures consistent and maintainable documentation across the project.

## Table of Contents
- [Package Documentation](#package-documentation)
- [Class/Interface Documentation](#classinterface-documentation)
- [Method Documentation](#method-documentation)
- [Field Documentation](#field-documentation)
- [Examples](#examples)

## Package Documentation

Package documentation should be provided in a file named `package-info.java` in the corresponding package directory.

```java
/**
 * This package contains utility classes for handling HTTP requests and responses.
 * 
 * <h2>Package Specification</h2>
 * <ul>
 *   <li>All classes in this package are thread-safe unless otherwise noted</li>
 *   <li>All exceptions thrown are documented at the class level</li>
 * </ul>
 * 
 * @since 1.0
 * @version 2.1
 */
package com.example.http;
```

## Class/Interface Documentation

Every class and interface should have a Javadoc comment that describes its purpose and usage.

Required tags:
- `@author`: Who wrote/maintains the class
- `@version`: Current version of the class
- `@since`: When the class was first added

Optional tags:
- `@see`: References to other related classes
- `@deprecated`: If the class is deprecated, include reason and alternative

```java
/**
 * A utility class that provides methods for parsing and formatting JSON data.
 * This class is thread-safe and can be used across multiple threads.
 *
 * <p>Example usage:</p>
 * <pre>
 * JsonParser parser = new JsonParser();
 * JsonObject obj = parser.parse("{\"name\": \"John\"}");
 * </pre>
 *
 * @author Jane Smith
 * @version 2.0
 * @since 1.0
 * @see JsonObject
 */
public class JsonParser {
    // Class implementation
}
```

## Method Documentation

Methods should be documented with:
- A clear description of what the method does
- All parameters
- Return value
- Any exceptions that can be thrown
- Usage examples for complex methods

Required tags:
- `@param`: For each parameter
- `@return`: What the method returns (except for void methods)
- `@throws`: For each checked exception

```java
/**
 * Parses the given JSON string and returns a JsonObject representation.
 * The parser handles nested objects and arrays.
 *
 * <p>Example:</p>
 * <pre>
 * String json = "{\"name\": \"John\", \"age\": 30}";
 * JsonObject obj = parseJson(json);
 * String name = obj.getString("name"); // Returns "John"
 * </pre>
 *
 * @param jsonString the JSON string to parse, must not be null
 * @param strict whether to enforce strict JSON syntax
 * @return a JsonObject representing the parsed JSON
 * @throws JsonParseException if the input is not valid JSON
 * @throws IllegalArgumentException if jsonString is null
 */
public JsonObject parseJson(String jsonString, boolean strict) throws JsonParseException {
    // Method implementation
}
```

## Field Documentation

Document fields that are part of the public API or have complex purposes.

```java
/**
 * The maximum number of concurrent connections allowed.
 * Must be between 1 and 100.
 */
public static final int MAX_CONNECTIONS = 10;
```

## Examples

### ✅ Good Examples

1. Well-documented class:
```java
/**
 * Represents a bank account with basic operations like deposit and withdraw.
 * This class is thread-safe and implements proper synchronization.
 *
 * <p>Example usage:</p>
 * <pre>
 * Account account = new Account("John Doe", 1000.0);
 * account.deposit(500.0);
 * account.withdraw(200.0);
 * </pre>
 *
 * @author Sarah Johnson
 * @version 1.1
 * @since 1.0
 */
public class Account {
    /**
     * Creates a new account with the specified owner and initial balance.
     *
     * @param owner the name of the account owner
     * @param initialBalance the starting balance
     * @throws IllegalArgumentException if initialBalance is negative
     */
    public Account(String owner, double initialBalance) {
        // Implementation
    }
}
```

2. Well-documented method:
```java
/**
 * Transfers money from this account to another account.
 * The transfer will only occur if this account has sufficient funds.
 *
 * @param destination the account to transfer to
 * @param amount the amount to transfer
 * @return true if the transfer was successful
 * @throws InsufficientFundsException if this account has insufficient funds
 * @throws IllegalArgumentException if amount is negative or destination is null
 */
public boolean transfer(Account destination, double amount) {
    // Implementation
}
```

### ❌ Bad Examples

1. Poor documentation (too vague):
```java
/**
 * Account class.
 */
public class Account {
    /**
     * Does the transfer.
     * @param dest destination
     * @param amt amount
     */
    public void transfer(Account dest, double amt) {
        // Implementation
    }
}
```

2. Missing important information:
```java
/**
 * Processes the data.
 */
public void processData(String input) {  // Missing @param, no mention of exceptions
    // Implementation
}
```

3. Redundant or obvious documentation:
```java
/**
 * This is the getName method that gets the name.
 * @return returns the name
 */
public String getName() {
    return name;
}
```

## Best Practices

1. Write documentation from the perspective of the API user
2. Document the "what" and "why" rather than the "how"
3. Include examples for non-obvious usage
4. Keep documentation up to date when code changes
5. Use complete sentences and proper grammar
6. Document any limitations or special considerations
7. Include units for numeric parameters/returns where applicable
8. Document thread-safety considerations
9. Use `{@code}` for inline code references
10. Use `{@link}` to reference other classes/methods

## Common Javadoc Tags

| Tag | Usage | Example |
|-----|--------|---------|
| `@author` | Class/interface author | `@author John Smith` |
| `@version` | Class/interface version | `@version 1.1` |
| `@param` | Method parameter | `@param name the user's name` |
| `@return` | Method return value | `@return the calculated sum` |
| `@throws` | Exception description | `@throws IllegalArgumentException if name is null` |
| `@since` | When feature was added | `@since 2.0` |
| `@see` | Reference to other element | `@see OtherClass#method` |
| `@deprecated` | Marks as deprecated | `@deprecated use newMethod() instead` |
| `{@code}` | Inline code reference | `Use {@code null} to reset` |
| `{@link}` | Link to other element | `See {@link OtherClass}` | 