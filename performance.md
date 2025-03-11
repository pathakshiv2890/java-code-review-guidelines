# Java Performance Optimization Guidelines

## 1. Data Structures

### Collection Selection
- Choose appropriate data structures based on access patterns
- Consider time complexity for common operations```java
// Random access needs
List<User> users = new ArrayList<>();  // O(1) access by index

// Frequent insertions/deletions
List<Task> tasks = new LinkedList<>();  // O(1) insert/delete at known position

// Key-based lookups
Map<String, User> userMap = new HashMap<>();  // O(1) average lookup
Map<String, User> concurrentMap = new ConcurrentHashMap<>();  // Thread-safe

// Sorted data
Map<Integer, Order> orderMap = new TreeMap<>();  // O(log n) operations, sorted keys
Set<String> uniqueSorted = new TreeSet<>();  // Sorted unique elements

// Fast contains/membership checks
Set<String> uniqueIds = new HashSet<>();  // O(1) contains
Set<String> concurrentSet = ConcurrentHashMap.newKeySet();  // Thread-safe

// Thread-safe collections
List<Task> syncList = Collections.synchronizedList(new ArrayList<>());
BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>(100);  // Bounded
```

### Collection Sizing
- Initialize collections with expected capacity when known
- Use specialized collections for memory efficiency
```java
// Preallocated capacity
List<String> items = new ArrayList<>(1000);  // Avoids incremental resizing
Map<String, User> userMap = new HashMap<>(1000, 0.75f);  // Initial capacity and load factor

// Memory efficient for primitives
int[] values = new int[1000];  // More efficient than Integer[]
IntStream.range(0, 100).forEach(i -> values[i] = i);
```

## 2. Streams and Parallel Processing

### Stream Operations
- Use streams for declarative and efficient data processing
- Choose appropriate terminal operations
```java
// Filtering and mapping
List<String> activeUserNames = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .collect(Collectors.toList());

// Reduction operations
double averageAge = users.stream()
    .mapToInt(User::getAge)
    .average()
    .orElse(0);

// Grouping and partitioning
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

### Parallel Processing
- Use parallel streams for CPU-intensive operations on large datasets
- Avoid parallel streams for I/O-bound or small datasets
```java
// Good use of parallel streams (CPU-intensive, large dataset)
long count = largeList.parallelStream()
    .filter(this::complexCalculation)
    .count();

// Bad use of parallel streams (I/O-bound)
list.parallelStream()
    .forEach(item -> databaseService.save(item));  // I/O bound, better sequential
```

## 3. Virtual Threads (Java 21)

### Thread Management
- Use virtual threads for I/O-bound operations
- Avoid virtual threads for CPU-intensive tasks
- Don't overuse virtual threads - use only when concurrent I/O operations are needed
```java
// Good - I/O bound tasks with virtual threads
@Service
public class EmailService {
    // Good - Multiple external API calls that can be parallelized
    public void sendBulkEmails(List<Email> emails) {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (Email email : emails) {
                executor.submit(() -> emailClient.send(email));  // I/O bound
            }
        }
    }

    // Bad - Simple operation doesn't need virtual threads
    public void sendSingleEmail(Email email) {
        Thread.startVirtualThread(() -> emailClient.send(email));  // Overkill for single operation
    }
}

// Bad - CPU intensive tasks with virtual threads
public void processData(List<Data> dataList) {
    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        for (Data data : dataList) {
            executor.submit(() -> {
                performHeavyComputation(data);  // CPU bound, use platform threads instead
            });
        }
    }
}

// Good - Simple sequential operation
public void simpleProcess() {
    // Don't use virtual threads for simple sequential operations
    processData();  // Regular method call is sufficient
}
```

### Structured Concurrency
- Use structured concurrency for managing virtual thread lifecycles
- Avoid manual thread management
```java
// Good - Structured concurrency
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<User> user = scope.fork(() -> userService.findUser(userId));
    Future<List<Order>> orders = scope.fork(() -> orderService.findOrders(userId));
    
    scope.join();           // Wait for all tasks
    scope.throwIfFailed();  // Propagate exceptions
    
    return new UserOrders(user.resultNow(), orders.resultNow());
}
```

## 4. Database Access Optimization
### Query Optimization
- Use named queries for frequently executed queries
- Use native queries for performance-critical operations
```java
// Named query (JPA)
@NamedQuery(
    name = "User.findActiveByDepartment",
    query = "SELECT u FROM User u WHERE u.active = true AND u.department = :dept"
)
public List<User> findActiveUsers(Department dept) {
    return entityManager.createNamedQuery("User.findActiveByDepartment", User.class)
        .setParameter("dept", dept)
        .getResultList();
}

// Native query for complex operations
@Query(value = "SELECT * FROM users u JOIN user_stats s ON u.id = s.user_id " +
               "WHERE s.login_count > :count AND u.status = 'ACTIVE'", 
       nativeQuery = true)
List<User> findPowerUsers(int count);
```

### Fetch Optimization
- Use pagination for large result sets
- Fetch only required data (avoid SELECT *)
- Use stream processing with pagination for large datasets

```java
// Basic pagination
Page<User> userPage = userRepository.findAll(
    PageRequest.of(0, 20, Sort.by("lastName").ascending())
);

// Stream processing with pagination for large datasets
public void processLargeDataset() {
    int pageSize = 1000;
    int pageNumber = 0;
    Page<Order> orderPage;
    
    do {
        orderPage = orderRepository.findAll(PageRequest.of(pageNumber++, pageSize));
        // Process each page
        orderPage.getContent().forEach(this::processOrder);
        
        // Clear persistence context to free memory
        entityManager.clear();
    } while (orderPage.hasNext());
}

// Using JPA Stream for very large datasets
@Transactional(readOnly = true)
public void processWithStream() {
    try (Stream<User> userStream = userRepository.streamAll()) {
        userStream
            .filter(User::isActive)
            .forEach(this::processUser);
    } // Stream automatically closed
}

// Projection for specific fields
@Query("SELECT new com.example.UserSummary(u.id, u.name, u.email) FROM User u")
List<UserSummary> findAllUserSummaries();
```

### Batch Operations
- Use batch inserts/updates for multiple operations
- Set appropriate batch size
```java
// Batch inserts with JdbcTemplate
jdbcTemplate.batchUpdate(
    "INSERT INTO orders (customer_id, amount) VALUES (?, ?)",
    new BatchPreparedStatementSetter() {
        public void setValues(PreparedStatement ps, int i) throws SQLException {
            ps.setLong(1, orders.get(i).getCustomerId());
            ps.setBigDecimal(2, orders.get(i).getAmount());
        }
        public int getBatchSize() {
            return orders.size();
        }
    }
);

// JPA batch configuration
@Bean
public JpaProperties jpaProperties() {
    JpaProperties props = new JpaProperties();
    props.getProperties().put("hibernate.jdbc.batch_size", "50");
    props.getProperties().put("hibernate.order_inserts", "true");
    return props;
}
```

## 5. Loops and Iterations

### Efficient Looping
- Avoid nested loops when possible
- Minimize work inside loops
```java
// Precompute outside loop
int size = list.size();
for (int i = 0; i < size; i++) {  // size() called only once
    // Process item
}

// Enhanced for loop for collections
for (User user : users) {  // Cleaner than iterator
    // Process user
}
```

### String Operations
- Use StringBuilder for string concatenation in loops
-  Avoid creating temporary strings
```java
// Good
StringBuilder sb = new StringBuilder();
for (String item : items) {
    sb.append(item);
}
String result = sb.toString();

// Bad
String result = "";
for (String item : items) {
    result += item;  // Creates new string each iteration
}
```

## 6. Resource Optimization

### Object Creation
- Minimize object creation in critical paths
- Reuse objects when appropriate
```java
// Reuse objects
private final DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE;

public String formatDate(LocalDate date) {
    return formatter.format(date);  // Reuses formatter
}

// Object pools for expensive objects
GenericObjectPool<ExpensiveObject> pool = new GenericObjectPool<>(factory);
try (var obj = pool.borrowObject()) {
    // Use the pooled object
}
```

### Memory Management
- Use primitive arrays instead of boxed types when possible
- Consider off-heap storage for very large datasets
```java
// Primitive arrays vs boxed collections
int[] primitiveArray = new int[1000];  // More efficient
Integer[] boxedArray = new Integer[1000];  // Less efficient

// Off-heap for large datasets
ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024);
// Use direct buffer for large data
``` 
