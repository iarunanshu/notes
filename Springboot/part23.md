### **Spring Boot JPA Part 3 | First-Level Caching | Notes**

---

### **Overview**
1. **First-Level Caching** is an integral feature of JPA/Hibernate that focuses on caching entities within the **Persistence Context** managed by **EntityManager** during the scope of a single HTTP request/session.
2. This caching mechanism avoids unnecessary database calls and ensures efficient querying within the lifecycle of an EntityManager/session.

---

### **Quick Recap of the Entity Lifecycle**
Before diving into first-level caching, understanding the **Entity Lifecycle** described in the previous part is essential:
1. **Transient State**:
    - New object created (e.g. `User obj = new User()`).
    - Not associated with database or persistence context.

2. **Managed State**:
    - Object is added to the persistence context when `EntityManager.persist()` is called.
    - Any modifications are tracked within the persistence context.
    - Stale DB reads are avoided if the entity already exists in persistence context.

3. **Removed State**:
    - Entity is marked for deletion in persistence context (`EntityManager.remove()`).
    - DB deletion occurs during flush/commit.

4. **Detached State**:
    - Entity is no longer managed by the persistence context (e.g., `EntityManager.close()` or `EntityManager.detach()`).

---

### **Introduction to First-Level Caching**
- **Persistence Context** acts as a first-level cache for entities within the scope of an **EntityManager**.
- Entities fetched through queries or persisted via `EntityManager.persist()` are stored in the persistence context.
- If a subsequent query requests the same entity, the persistence context returns the cached entity instead of querying the database.

---

### **Characteristics and Behavior**
1. **Scope**:
    - Limited to the scope of **EntityManager**.
    - One EntityManager maps to one Persistence Context.
    - Multiple EntityManager instances have separate, independent Persistence Contexts.

2. **Isolation**:
    - Persistence contexts of different EntityManager instances are completely isolated and do not share cached entities.

3. **Automatic Caching**:
    - Frequently accessed or modified entities are automatically stored in the persistence context without manual intervention.

4. **Flush and Commit**:
    - Changes in persistence context are synchronized with the database upon **flush** or **commit**.

5. **HTTP Request Lifecycle**:
    - In typical Spring Boot applications, a new EntityManager is created for each HTTP request, and the persistence context holds entities for that session.

---

### **Example: First-Level Caching**

#### **Setup**
- **Controller**:
  ```java
  @RestController
  public class UserController {
      @Autowired
      private UserService userService;

      @GetMapping("/test-jpa")
      public ResponseEntity<?> testJpaCaching() {
          userService.saveUser(new UserDetails("John", "john@example.com"));
          UserDetails user = userService.getUserById(1L); // First access
          return ResponseEntity.ok(user);
      }

      @GetMapping("/read-jpa")
      public ResponseEntity<?> readJpaCaching() {
          UserDetails user = userService.getUserById(1L); // Another HTTP request - New context
          return ResponseEntity.ok(user);
      }
  }
  ```

- **Service**:
  ```java
  @Service
  public class UserService {
      @Autowired
      private UserDetailsRepository userDetailsRepository;

      @Transactional
      public void saveUser(UserDetails user) {
          userDetailsRepository.save(user);
      }

      public UserDetails getUserById(Long id) {
          return userDetailsRepository.findById(id).orElse(null); // Fetch user entity
      }
  }
  ```

#### **Runtime Behavior**
1. **Single HTTP Request**:
    - EntityManager persists the new user (`saveUser`) into DB using `persist()`, and the object is added into the persistence context.
    - When `getUserById()` is called:
        - The persistence context is checked first.
        - No SQL SELECT query is fired because the entity already exists in the persistence context from `persist()`.

2. **Separate HTTP Request**:
    - A new HTTP request creates a new EntityManager.
    - A fresh persistence context is created (different from the previous request).
    - `getUserById()` queries the database and loads the entity into this persistence context because the entity is not present in its cache.
    - Only one SQL SELECT query is fired.

---

### **Detailed Walkthrough: Internal Flow**
1. **Insert Scenario**:
    - When `EntityManager.persist()` is called, the entity is added into the persistence context.
    - An explicit SQL **INSERT** query is triggered during **flush** or **commit**.

2. **Fetch Scenario (First-Level Cache in Action)**:
    - When `EntityManager.find()` is called:
        - The persistence context first checks if the entity with the requested primary key already exists in its cache.
        - If found, it returns the cached entity (NO DB query occurs).
        - If not found, a SELECT SQL query is fired to load the entity from the DB, and it is stored in the persistence context.

3. **Working Across Multiple HTTP Requests**:
    - Each HTTP request has its own EntityManager and Persistence Context.
    - If an entity was cached in Persistence Context during the first request, it cannot be reused by the second request since the EntityManagers are isolated.

---

### **Code Walkthrough Debugging**
#### **Insert and Fetch (Single HTTP Request)**
- Steps:
    1. **Insert**: Entity added to Persistence Context via `EntityManager.persist()`.
        - Example insertion of user:
        ```
        Insert query generated:
        INSERT INTO user_details (name, email) VALUES ('John', 'john@example.com');
        ```
    2. **Fetch**:
        - Lookup inside persistence context:
        - No SELECT query fired—data returned from cache.

#### **Fetch in a New HTTP Request**
- Steps with new EntityManager:
    1. **Lookup**: Persistence Context doesn’t contain the requested entity.
    2. **Fetch from DB**:
       ```
       SELECT query generated:
       SELECT * FROM user_details WHERE id = 1;
       ```
    3. **Entity Added**: Fetched entity stored in new Persistence Context.

---

### **Why Cache Works?**
- Spring Boot creates an EntityManager instance for each HTTP request.
- EntityManager uses its Persistence Context to cache entities within the lifecycle of the HTTP request.
- Queries are optimized by preventing redundant DB calls when cached data exists.

---

### **Key Observations**
1. **Insert**:
    - `EntityManager.persist()` triggers an insert query only during **flush/commit**.

2. **Find/Fetched Entity**:
    - If the Persistence Context contains the entity, it is directly returned (cache hit).
    - Saves DB access and improves application efficiency.

3. **Context Isolation**:
    - First-level cache respects EntityManager isolation—entities in one context cannot be accessed by another.

---

### **Scenarios that Trigger SQL Queries**
1. **Entity Not Found in Persistence Context**: Select queries hit the DB.
2. **New HTTP Request**:
    - A new EntityManager and Persistence Context trigger a fresh SQL query unless external caching mechanisms are used (such as second-level caching).

---

### **Advantages of First-Level Caching**
1. **Performance Optimization**:
    - Reduces redundant DB calls by checking persistence context first.
2. **Managed Entity Lifecycle**:
    - Tracks entities and synchronizes their state with the database.

3. **Session Efficiency**:
    - Enhances efficiency by limiting the scope of caching to the HTTP request lifecycle.

---

### **Limitations/Next Steps**
1. **First-Level Cache Scope**:
    - Limited to the lifespan of a single EntityManager/session (cannot share cached entities across sessions).

2. **Second-Level Caching**:
    - Next step when caching is required across multiple requests.
    - Hibernate offers second-level caching to supplement first-level caching.

---

### **Summary**
1. First-level caching is performed by the **Persistence Context** associated with an **EntityManager**.
2. Cached entities are available for CRUD operations within a single HTTP request/session lifecycle.
3. Each new HTTP request/session creates a fresh EntityManager, leading to new Persistence Context and isolation.
4. It improves application efficiency by preventing redundant database queries.

Up next: **Second-Level Caching**! Feel free to ask questions to clarify anything further. 😊