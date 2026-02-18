# Spring Boot Security - User Creation (Part 2) - Detailed Notes

## 1. Introduction

### Why User Creation First?
> Authentication and Authorization of user will happen **only after user is created**

Before learning authentication methods (Form Login, Basic Auth, JWT, OAuth), we must understand:
- How to create users
- Production-level user creation best practices

---

## 2. Default Behavior When Adding Security Dependency

### Adding the Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### What Happens When Server Starts?

When you start the application, logs show:
```
Using generated security password: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Using in-memory user detail manager
```

### Default User Creation
Spring Boot Security **automatically creates a user**:

| Property | Default Value |
|----------|--------------|
| Username | `user` |
| Password | Random UUID (printed in logs) |
| Roles | Empty |

**Important**: Each time server restarts → New random password generated

### Internal Working

#### Step 1: SecurityProperties.java (Spring Framework)
```java
public class SecurityProperties {
    
    public static class User {
        private String name = "user";                    // Default username
        private String password = UUID.randomUUID().toString();  // Random password
        private List<String> roles = new ArrayList<>();  // Empty roles
        
        // Getters and Setters
    }
}
```

#### Step 2: UserDetailsServiceAutoConfiguration.java
Creates a bean that:
1. Takes `SecurityProperties.User` object
2. Maps it to `UserDetails` object
3. Creates `InMemoryUserDetailsManager` with this user

```java
@Bean
public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties) {
    SecurityProperties.User user = properties.getUser();
    
    UserDetails userDetails = User.builder()
        .username(user.getName())
        .password(user.getPassword())
        .roles(user.getRoles().toArray(new String[0]))
        .build();
    
    return new InMemoryUserDetailsManager(userDetails);
}
```

#### Step 3: InMemoryUserDetailsManager
```java
public class InMemoryUserDetailsManager {
    
    private final Map<String, UserDetails> users = new HashMap<>();  // In-memory storage
    
    public InMemoryUserDetailsManager(UserDetails... users) {
        for (UserDetails user : users) {
            createUser(user);
        }
    }
    
    public void createUser(UserDetails user) {
        users.put(user.getUsername(), user);  // Store in memory
    }
}
```

### Class Hierarchy
```
UserDetailsService (Interface)
        △
        │
UserDetailsManager (Interface)
        △
        │
InMemoryUserDetailsManager (Class)
```

---

## 3. Approach 1: Application.properties

### Configuration
```properties
spring.security.user.name=my_username
spring.security.user.password=my_password
spring.security.user.roles=ADMIN,USER
```

### How It Works
- Spring Boot uses **reflection** to call setter methods
- Overrides default values in `SecurityProperties.User`

```java
// Internally, Spring calls:
securityProperties.getUser().setName("my_username");
securityProperties.getUser().setPassword("my_password");
securityProperties.getUser().setRoles(Arrays.asList("ADMIN", "USER"));
```

### Result
- No default password generated in logs
- Custom username/password used

### Limitations
❌ **Only ONE user** can be created
❌ **Not recommended for production**
✅ Only for development/testing

---

## 4. Approach 2: Custom InMemoryUserDetailsManager Bean

### Why This Approach?
- Create **multiple users**
- Understand internal working
- More control over user creation

### Implementation

```java
@Configuration
public class SecurityConfig {
    
    @Bean
    public UserDetailsService userDetailsService() {
        
        // Create User 1
        UserDetails user1 = User.builder()
            .username("my_username_1")
            .password("{noop}my_password_1")  // Note the {noop}
            .roles("USER")
            .build();
        
        // Create User 2
        UserDetails user2 = User.builder()
            .username("my_username_2")
            .password("{noop}1234")
            .roles("ADMIN")
            .build();
        
        // Return InMemoryUserDetailsManager with multiple users
        return new InMemoryUserDetailsManager(user1, user2);
    }
}
```

### Why Use Parent Interface?
```
UserDetailsService (Interface)     ← We return this
        △
        │
UserDetailsManager (Interface)
        △
        │
InMemoryUserDetailsManager (Class) ← We create this
```

**Best Practice**: Return parent interface type for flexibility

---

## 5. Password Storage Format

### Default Format
```
{id}encodedPassword
```

| ID | Meaning | Example |
|----|---------|---------|
| `{noop}` | No encoding/hashing (plain text) | `{noop}1234` |
| `{bcrypt}` | BCrypt hashing (one-way, secure) | `{bcrypt}$2a$10$...` |
| `{sha256}` | SHA-256 algorithm | `{sha256}abc123...` |

### What is `{noop}`?
- **No Operation** = No encoding, no hashing
- Password stored as **plain text**
- Only for testing, **NEVER for production**

```java
.password("{noop}my_plain_password")
```

### How Password Validation Works

#### Scenario: User stored with `{bcrypt}`
```
Stored in DB: {bcrypt}$2a$10$xyz123abc...
User enters: 1234
```

#### Flow:
```
Step 1: Fetch user from DB/memory by username
        ↓
Step 2: Extract password → {bcrypt}$2a$10$xyz123abc...
        ↓
Step 3: Read ID → bcrypt
        ↓
Step 4: Spring knows: "Password is hashed with BCrypt"
        ↓
Step 5: Hash user's input (1234) using BCrypt
        ↓
Step 6: Compare two hashes
        ↓
Step 7: If match → Authenticated ✓
```

### DelegatingPasswordEncoder

```
User Password Input
        ↓
DelegatingPasswordEncoder (Default)
        ↓
    Reads {id} from stored password
        ↓
    ┌─────┴─────────┐
    ↓               ↓
{noop}          {bcrypt}
    ↓               ↓
NoOpEncoder    BCryptEncoder
    ↓               ↓
Direct Compare  Hash & Compare
```

> **DelegatingPasswordEncoder** = Delegates to appropriate encoder based on ID

### Storing Hashed Password

```java
@Bean
public UserDetailsService userDetailsService() {
    
    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    
    UserDetails user = User.builder()
        .username("my_username")
        .password("{bcrypt}" + encoder.encode("my_password_1"))  // Hash it!
        .roles("USER")
        .build();
    
    return new InMemoryUserDetailsManager(user);
}
```

---

## 6. Specifying Password Encoder Explicitly

### Problem
- Don't want to store `{bcrypt}` prefix in password
- Want cleaner password storage

### Solution: Create PasswordEncoder Bean

```java
@Configuration
public class SecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();  // Always use BCrypt
    }
    
    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder encoder) {
        
        UserDetails user = User.builder()
            .username("my_username")
            .password(encoder.encode("1234"))  // No {bcrypt} prefix needed!
            .roles("USER")
            .build();
        
        return new InMemoryUserDetailsManager(user);
    }
}
```

### How It Works
- When `PasswordEncoder` bean exists → Spring uses it directly
- No delegation needed
- No `{id}` prefix required

---

## 7. Approach 3: Database Storage (Production Recommended)

### Why DB Approach?
| In-Memory | Database |
|-----------|----------|
| Lost on restart | Persistent |
| Static users | Dynamic user registration |
| Not scalable | Scalable |
| Testing only | Production ready |

### Architecture Overview

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Controller     │────▶│     Service      │────▶│   Repository     │
│ (Register API)   │     │ (UserAuthService)│     │(UserAuthEntityRepo)│
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                                           │
                                                           ▼
                                                  ┌──────────────────┐
                                                  │    Database      │
                                                  │  (user_auth)     │
                                                  └──────────────────┘
```

### Step 1: Create Entity

```java
@Entity
@Table(name = "user_auth")
public class UserAuthEntity implements UserDetails {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String password;
    private String roles;
    
    // Why implement UserDetails?
    // During authentication, Spring expects UserDetails object
    // By implementing it, no mapping needed!
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.stream(roles.split(","))
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
    }
    
    @Override
    public String getPassword() {
        return password;
    }
    
    @Override
    public String getUsername() {
        return username;
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return true;  // Custom logic in production
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return true;  // Custom logic in production
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true;  // Custom logic in production
    }
    
    @Override
    public boolean isEnabled() {
        return true;  // Custom logic in production
    }
    
    // Getters and Setters
}
```

### Why Implement UserDetails?

**Without UserDetails Implementation:**
```java
// Must do mapping manually
public UserDetails loadUserByUsername(String username) {
    UserAuthEntity entity = repository.findByUsername(username);
    
    // Manual mapping required!
    return User.builder()
        .username(entity.getUsername())
        .password(entity.getPassword())
        .roles(entity.getRoles())
        .build();
}
```

**With UserDetails Implementation:**
```java
// No mapping needed!
public UserDetails loadUserByUsername(String username) {
    return repository.findByUsername(username);  // Direct return
}
```

### Step 2: Create Repository

```java
@Repository
public interface UserAuthEntityRepository extends JpaRepository<UserAuthEntity, Long> {
    
    Optional<UserAuthEntity> findByUsername(String username);
    // Spring auto-generates: SELECT * FROM user_auth WHERE username = ?
}
```

### Step 3: Create Service (Implements UserDetailsService)

```java
@Service
public class UserAuthEntityService implements UserDetailsService {
    
    @Autowired
    private UserAuthEntityRepository repository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    // Called by Spring during authentication
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return repository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    }
    
    // Called during registration
    public UserAuthEntity save(UserAuthEntity user) {
        return repository.save(user);
    }
}
```

### Why Implement UserDetailsService?

```
Authentication Flow:
    ↓
Spring Framework calls loadUserByUsername()
    ↓
┌─────────────────┐
│ For In-Memory:  │ → InMemoryUserDetailsManager.loadUserByUsername()
│ Spring knows    │    (Fetches from HashMap)
└─────────────────┘
    ↓
┌─────────────────┐
│ For Database:   │ → OUR UserAuthEntityService.loadUserByUsername()
│ Spring doesn't  │    (We tell Spring how to fetch!)
│ know which table│
└─────────────────┘
```

### Class Hierarchy After Implementation

```
UserDetailsService (Interface)
        △
   ┌────┴────────────────┐
   │                     │
UserDetailsManager    UserAuthEntityService
   △                  (Our Custom Service)
   │
InMemoryUserDetailsManager
```

### Step 4: Create Controller

```java
@RestController
@RequestMapping("/auth")
public class AuthController {
    
    @Autowired
    private UserAuthEntityService userService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @PostMapping("/register")
    public ResponseEntity<String> register(@RequestBody UserAuthEntity user) {
        
        // IMPORTANT: Hash password before saving!
        String encodedPassword = passwordEncoder.encode(user.getPassword());
        user.setPassword(encodedPassword);
        
        userService.save(user);
        
        return ResponseEntity.ok("User registered successfully");
    }
}
```

### Step 5: Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                // BYPASS authentication for register API
                .requestMatchers("/auth/register").permitAll()
                // All other APIs require authentication
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
}
```

### Why Bypass /auth/register?
- This is the API to CREATE username/password
- User doesn't HAVE credentials yet
- Must be accessible without authentication
- **Industry standard practice**

---

## 8. Complete Flow Summary

### User Registration Flow

```
POST /auth/register
{username: "u1", password: "1234"}
        ↓
AuthController.register()
        ↓
Hash password: "1234" → "$2a$10$xyz..."
        ↓
UserAuthEntityService.save()
        ↓
Repository saves to DB
        ↓
Database:
┌────┬──────────┬─────────────────┬───────┐
│ ID │ USERNAME │    PASSWORD     │ ROLES │
├────┼──────────┼─────────────────┼───────┤
│  1 │    u1    │ $2a$10$xyz...   │ USER  │
└────┴──────────┴─────────────────┴───────┘
```

### User Authentication Flow

```
GET /api/protected
Authorization: Basic dTE6MTIzNA==  (u1:1234)
        ↓
Spring Security Filter
        ↓
Calls UserAuthEntityService.loadUserByUsername("u1")
        ↓
Repository.findByUsername("u1")
        ↓
Returns UserAuthEntity (implements UserDetails)
        ↓
PasswordEncoder compares:
  - User input: "1234" → Hash it → "$2a$10$abc..."
  - DB stored: "$2a$10$xyz..."
        ↓
If match → Access granted ✓
If not → 401 Unauthorized ✗
```

---

## 9. Production Considerations

### UserDetails Override Methods

| Method | Purpose | Production Logic Example |
|--------|---------|-------------------------|
| `isAccountNonExpired()` | Account validity | Check expiry date column |
| `isAccountNonLocked()` | Account lock status | Check after N failed attempts |
| `isCredentialsNonExpired()` | Password validity | Password > 90 days old? |
| `isEnabled()` | Account active | Email verified? |

### Example: Credentials Expiry Logic
```java
@Column(name = "password_updated_at")
private LocalDateTime passwordUpdatedAt;

@Override
public boolean isCredentialsNonExpired() {
    // Password expires after 90 days
    return passwordUpdatedAt.plusDays(90).isAfter(LocalDateTime.now());
}
```

---

## 10. Summary: Three Approaches Compared

| Aspect | Application.properties | In-Memory Bean | Database |
|--------|----------------------|----------------|----------|
| Multiple Users | ❌ No | ✅ Yes | ✅ Yes |
| Persistent | ❌ No | ❌ No | ✅ Yes |
| Dynamic Registration | ❌ No | ❌ No | ✅ Yes |
| Password Hashing | ⚠️ Manual | ⚠️ Manual | ✅ Yes |
| Production Ready | ❌ No | ❌ No | ✅ Yes |
| Use Case | Dev/Testing | Learning/Testing | Production |

---

## 11. Key Takeaways

1. **User creation is prerequisite** for authentication/authorization
2. **Default behavior**: Spring creates `user` with random password
3. **Password format**: `{id}encodedPassword` (e.g., `{bcrypt}$2a$...`)
4. **DelegatingPasswordEncoder**: Routes to correct encoder based on `{id}`
5. **For production**: Use Database approach with:
    - Entity implementing `UserDetails`
    - Service implementing `UserDetailsService`
    - Password hashing with `BCryptPasswordEncoder`
    - Bypass authentication for registration API