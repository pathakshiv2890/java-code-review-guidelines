# Spring Framework Best Practices (Java 21, Spring 3.4.1)

## 1. Dependency Injection

### Constructor Injection
- Prefer constructor injection over field injection
- Make dependencies final
- Use `@RequiredArgsConstructor` for clean constructor injection
```java
// Good
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
}

// Bad
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
}
```

### Component Stereotypes
- Use appropriate stereotypes
```java
@RestController  // For REST endpoints
@Controller      // For MVC controllers
@Service        // For business logic
@Repository     // For data access
@Component      // For general components
@Configuration  // For configuration classes
```

## 2. Transaction Management

### Transaction Annotations
- Use `@Transactional` at service level
- Specify read-only when applicable
- Define proper propagation levels
```java
// Good
@Transactional(readOnly = true)
public List<User> getAllUsers() {
    return userRepository.findAll();
}

@Transactional(propagation = Propagation.REQUIRED)
public User createUser(UserDTO dto) {
    // implementation
}
```

### Transaction Boundaries
- Keep transactions as short as possible
- Don't do heavy processing within transactions
- Handle exceptions properly
```java
@Transactional
public void processOrder(Order order) {
    // Do data validation outside transaction
    validateOrder(order);
    
    // Only transactional operations here
    order = orderRepository.save(order);
    updateInventory(order);
    
    // Notifications/emails should be outside transaction
    notifyCustomer(order);
}
```

## 3. Virtual Threads (Java 21)

- Use Virtual Threads for I/O-intensive operations


### Task Execution
- Configure Spring's `TaskExecutor` to use Virtual Threads if needed
- Use for asynchronous tasks
```java
@Configuration
public class AsyncConfig {
    @Bean
    public AsyncTaskExecutor applicationTaskExecutor() {
        return new TaskExecutorAdapter(Executors.newVirtualThreadPerTaskExecutor());
    }
}
```

### Best Practices for Virtual Threads
- Avoid thread-local variables with long-lived threads
- Don't use Virtual Threads for CPU-intensive tasks
- Prefer `@Async` over manual thread management
```java
// Good
@Service
public class EmailService {
    @Async
    public CompletableFuture<Boolean> sendEmailAsync(String to, String subject) {
        // Email sending logic
        return CompletableFuture.completedFuture(true);
    }
}

// Bad - Manual thread management
public void sendEmails(List<String> recipients) {
    for (String recipient : recipients) {
        Thread.startVirtualThread(() -> {
            sendEmail(recipient);
        });
    }
}
```

## 4. Caching

### Cache Configuration
- Use meaningful cache names
- Set appropriate TTL
- Use cache conditions
```java
@Cacheable(
    value = "users",
    key = "#id",
    unless = "#result == null",
    cacheManager = "userCacheManager"
)
public User findById(Long id) {
    return userRepository.findById(id);
}
```

### Cache Eviction
- Clear cache on updates
- Use appropriate eviction strategy
```java
@CacheEvict(value = "users", key = "#user.id")
public void updateUser(User user) {
    userRepository.save(user);
}

@CacheEvict(value = "users", allEntries = true)
@Scheduled(fixedRateString = "${cache.evict.rate:3600000}")
public void evictAllCaches() {
    // Cache eviction logic
}
```

## 5. REST API Design

### Controller Structure
- Use appropriate HTTP methods
- Return proper status codes
- Handle exceptions globally
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody UserDTO dto) {
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(userService.createUser(dto));
    }
}
```

### Request/Response DTOs
- Use DTOs for request/response
- Validate DTOs using Bean Validation
```java
@Data
public class UserDTO {
    @NotNull
    @Size(min = 2, max = 50)
    private String name;
    
    @Email
    private String email;
    
    @NotBlank
    @Pattern(regexp = "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{8,}$")
    private String password;
}

## 6. Project Structure

### Multi-Module Architecture
- Our project follows a modular architecture with separate API and Service modules

### API Module
- Contains all models, DTOs, and REST interfaces
- Defines the contract for external consumers
- Includes request/response wrappers
- Contains OpenAPI/Swagger documentation

```java
// API Module Structure Example
api-module/
  ├── src/main/java/
  │   └── com/company/project/
  │       ├── api/
  │       │   ├── UserApi.java         // REST interface with @RequestMapping
  │       │   └── OrderApi.java
  │       ├── model/
  │       │   ├── UserDTO.java         // Data transfer objects
  │       │   └── OrderDTO.java
  │       └── wrapper/
  │           ├── ResponseWrapper.java  // Standard response format
  │           └── PagedResponse.java
  └── pom.xml
```

### Service Module
- Contains service interfaces and implementations
- Includes DAO (Data Access Object) interfaces and implementations
- Houses REST implementation classes
- Implements business logic

```java
// Service Module Structure Example
service-module/
  ├── src/main/java/
  │   └── com/company/project/
  │       ├── rest/
  │       │   ├── UserApiImpl.java     // Implements UserApi interface
  │       │   └── OrderApiImpl.java
  │       ├── service/
  │       │   ├── UserService.java     // Service interface
  │       │   ├── UserServiceImpl.java // Service implementation
  │       │   ├── OrderService.java
  │       │   └── OrderServiceImpl.java
  │       └── dao/
  │           ├── UserDao.java         // DAO interface
  │           ├── UserDaoImpl.java     // DAO implementation
  │           ├── OrderDao.java
  │           └── OrderDaoImpl.java
  └── pom.xml
```

### Best Practices for This Structure
- Keep API interfaces clean and focused on contract definition
- Implement proper separation of concerns between layers
- Use dependency injection to wire components together
- Ensure service implementations are properly tested

```java
// API Interface Example
@RequestMapping("/api/v1/users")
public interface UserApi {
    @GetMapping("/{id}")
    ResponseEntity<ResponseWrapper<UserDTO>> getUser(@PathVariable Long id);
    
    @PostMapping
    ResponseEntity<ResponseWrapper<UserDTO>> createUser(@Valid @RequestBody UserDTO dto);
}

// REST Implementation Example
@RestController
public class UserApiImpl implements UserApi {
    private final UserService userService;
    
    public UserApiImpl(UserService userService) {
        this.userService = userService;
    }
    
    @Override
    public ResponseEntity<ResponseWrapper<UserDTO>> getUser(Long id) {
        UserDTO user = userService.findById(id);
        return ResponseEntity.ok(new ResponseWrapper<>(user));
    }
    
    @Override
    public ResponseEntity<ResponseWrapper<UserDTO>> createUser(UserDTO dto) {
        UserDTO created = userService.createUser(dto);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(new ResponseWrapper<>(created));
    }
}
