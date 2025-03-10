# Java Memory Leak Prevention Guidelines

## 1. Resource Management

### Closeable Resources
- ✅ Always close resources (connections, streams, files)
- ✅ Use try-with-resources for all AutoCloseable objects
```java
// Good
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(SQL)) {
    // Use resources
}

// Bad
Connection conn = dataSource.getConnection();
// No closing mechanism
```

### Thread Management
- ✅ Properly shut down thread pools and executors
- ✅ Avoid creating threads without lifecycle management
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

## 2. Reference Management

### Collection Memory Leaks
- ✅ Clear collections when no longer needed
- ✅ Be careful with long-lived HashMaps and Lists
- ✅ Remove items from collections after processing
- ✅ Use appropriate collection sizes and types

```java
// Bad - Memory leak in HashMap
public class UserCache {
    private static final Map<String, User> userMap = new HashMap<>();
    
    public void addUser(String id, User user) {
        userMap.put(id, user);  // Map keeps growing, never cleared
    }
}

// Good - Proper HashMap management
public class UserCache {
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

// Bad - Memory leak in List
public class OrderProcessor {
    private List<Order> processedOrders = new ArrayList<>();
    
    public void processOrder(Order order) {
        // Process the order
        processedOrders.add(order);  // List keeps growing
    }
}

// Good - Proper List management
public class OrderProcessor {
    private List<Order> currentOrders = new ArrayList<>();
    
    public void processOrder(Order order) {
        currentOrders.add(order);
        // Process the order
        currentOrders.remove(order);  // Remove after processing
    }
    
    // For batch processing
    public void processBatch(List<Order> orders) {
        logger.info("Processing batch of {} orders", orders.size());
        // Process orders
        currentOrders.clear();  // Clear after batch processing
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


```

## 3. Caching Strategies

### Cache Implementation
- ✅ Use WeakHashMap for memory-sensitive caches
- ✅ Implement size limits and eviction policies
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

### Listener Management
- ✅ Unregister listeners and callbacks when no longer needed
- ✅ Use weak references for long-lived listeners
```java
// Good
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        // Handle event
    }
});
// Later
button.removeActionListener(listener);

// Bad
someService.addListener(this); // Never removed
```

## 4. Common Patterns to Avoid

### Inner Class References
- ✅ Be careful with non-static inner classes (they hold reference to outer class)
- ✅ Use static inner classes when outer class reference isn't needed
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
- ✅ Always clean up ThreadLocal variables
- ✅ Use try-finally to ensure cleanup
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