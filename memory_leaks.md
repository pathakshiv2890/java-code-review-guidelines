# Java Memory Leak Prevention Guidelines

## 1. Resource Management

### Closeable Resources
-  Always close resources (connections, streams, files)
-  Use try-with-resources for all AutoCloseable objects
```java
// Good - Resources automatically closed
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(SQL);
     ResultSet rs = stmt.executeQuery()) {
    // Use resources
    while (rs.next()) {
        // Process results
    }
} // All resources automatically closed here

// Bad - Resources not properly closed
Connection conn = null;
PreparedStatement stmt = null;
ResultSet rs = null;
try {
    conn = dataSource.getConnection();
    stmt = conn.prepareStatement(SQL);
    rs = stmt.executeQuery();
    // Use resources
} catch (SQLException e) {
    throw new DatabaseException("Query failed", e);
} finally {
    // Manual closing - error-prone and verbose
    if (rs != null) try { rs.close(); } catch (SQLException e) { }
    if (stmt != null) try { stmt.close(); } catch (SQLException e) { }
    if (conn != null) try { conn.close(); } catch (SQLException e) { }
}
```

### Thread Management
- Properly shut down thread pools and executors
- Avoid creating threads without lifecycle management
```java
// Good
ExecutorService executor = Executors.newFixedThreadPool(10);
try {
    // Use executor
} finally {
    executor.shutdown();
}

// Bad
ExecutorService executor = Executors.newFixedThreadPool(10);
// No shutdown
```

## 2. Collection Memory Leaks

### Collection Memory Leaks
- Clear collections when no longer needed
- Be careful with long-lived HashMaps and Lists
- Remove items from collections after processing
- Use appropriate collection sizes and types

```java
// Bad - Memory leak in static HashMap
public class UserCache {
    // Static collections should only be used when data truly needs to be shared across all instances
    // For instance-level caching, use instance variables instead
    private static final Map<String, User> userMap = new HashMap<>();
    
    public void addUser(String id, User user) {
        userMap.put(id, user);  // Map keeps growing, never cleared
    }
}

// Good - Instance level caching with proper management
public class UserCache {
    // Instance-level map - each instance has its own cache
    private final Map<String, User> userMap = new HashMap<>();
    
    public void addUser(String id, User user) {
        userMap.put(id, user);
    }
    
    public void removeUser(String id) {
        userMap.remove(id);  // Remove when no longer needed
    }
    
    public void clearCache() {
        userMap.clear();  // Clear all entries when needed
    }
}

// Good - Static cache with proper management (when sharing across instances is required)
public class GlobalUserCache {
    // Static cache with size limit and cleanup mechanism
    private static final int MAX_ENTRIES = 1000;
    private static final Map<String, User> globalUserMap = new LinkedHashMap<>(MAX_ENTRIES + 1, .75F, true) {
        @Override
        protected boolean removeEldestEntry(Map.Entry<String, User> eldest) {
            return size() > MAX_ENTRIES;
        }
    };
    
    public static void addUser(String id, User user) {
        globalUserMap.put(id, user);  // Automatically removes oldest entry if size exceeds MAX_ENTRIES
    }
}

### Common Collection Memory Leak Scenarios

1. **Never removing items**:
   - Adding items to a collection but never removing them
   - Collections growing indefinitely over application lifetime

2. **Forgotten references**:
   - Keeping references to objects in collections after they're no longer needed
   - Not clearing temporary processing collections

3. **Static collections**:
   - Using static collections that live for entire application lifetime
   - Not managing the size of long-lived collections

## 3. Caching Strategies

### Cache Implementation
- Use WeakHashMap for memory-sensitive caches
- Implement size limits and eviction policies
```java
// Good
LoadingCache<Key, Value> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(new CacheLoader<Key, Value>() {
        public Value load(Key key) throws Exception {
            return createValue(key);
        }
    });

// Bad
private Map<String, ExpensiveObject> cache = new HashMap<>(); // Unbounded
```

## 4. Common Patterns to Avoid

### Inner Class References
- Be careful with non-static inner classes (they hold reference to outer class)
-  Use static inner classes when outer class reference isn't needed
```java
// Good
private static class StaticHelper {
    void help() { /* ... */ }
}

// Bad - Holds reference to enclosing instance
private class NonStaticHelper {
    void help() { /* ... */ }
}
```

### ThreadLocal Usage
- Always clean up ThreadLocal variables
- Use try-finally to ensure cleanup
```java
// Good
ThreadLocal<Resource> resourceHolder = new ThreadLocal<>();
try {
    resourceHolder.set(new Resource());
    // Use resource
} finally {
    resourceHolder.remove(); // Clean up
}

// Bad
ThreadLocal<Resource> resourceHolder = new ThreadLocal<>();
resourceHolder.set(new Resource());
// No cleanup
``` 