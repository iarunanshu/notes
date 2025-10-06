Here's a summarized set of notes based on your transcript:

---

### **Spring Boot JDBC Part 1 | Notes**

---

### **Overview**
1. JDBC (Java Database Connectivity) is used to connect to a relational database, query it, and process results.
2. JPA is a framework that abstracts the JDBC layer, allowing interactions with relational databases using Java objects instead of SQL queries.
3. This session focuses on JDBC ((Java Database Connectivity)) implementation in plain Java and how Spring Boot simplifies JDBC through the **JDBC Template**.

---

### **JDBC Architecture**
- **Sequence of Interaction**:
    1. **Application Logic**: Sends requests.
    2. **JPA** (if used): Provides API, but only as interfaces (no implementations).
    3. **ORM Frameworks** (e.g., Hibernate, OpenJPA, EclipseLink): Implements JPA and interacts with JDBC.
    4. **JDBC APIs**: Provides standard interfaces (e.g., `getConnection`, `executeQuery`, etc.).
    5. **DB Drivers**: Implements JDBC APIs for specific databases (e.g., **MySQL Connector/J** or PostgreSQL JDBC driver).

- **JDBC Roles**:
    - *JDBC API*: Standard interface for database interactions.
    - *Drivers*: Actual implementation for specific databases (e.g., MySQL, PostgreSQL, H2).

---

### **Steps in JDBC Without Spring Boot**
#### a) **Plain JDBC Implementation**
##### Example Setup:
- Driver for H2 Database (`org.h2.Driver`):
    - **Driver Loading**: Use `Class.forName(driverClassName)`.
    - **Establish Connection**: `DriverManager.getConnection(url, user, password)` (e.g., `jdbc:h2:mem:userdb`).
    - **Query Execution**:
        - Use `PreparedStatement` for SQL execution.
        - Use `ResultSet` for fetching results.
    - **Closing Resources**:
        - Always close `Connection`, `Statement`, and `ResultSet` to avoid resource leakage.

##### Example Methods:
1. **Create Table in DB**:
    ```java
    String query = "CREATE TABLE users (userId INT AUTO_INCREMENT, username VARCHAR(255), age INT)";
    statement.executeUpdate(query);
    ```
2. **Insert Data**:
    ```java
    String query = "INSERT INTO users (username, age) VALUES (?, ?)";
    preparedStatement.setString(1, "name");
    preparedStatement.setInt(2, age);
    preparedStatement.executeUpdate();
    ```
3. **Read Data**:
    ```java
    String query = "SELECT * FROM users";
    ResultSet resultSet = preparedStatement.executeQuery(query);
    while (resultSet.next()) {
        // Process rows
    }
    ```

##### **Issues/Challenges with Plain JDBC**:
1. **Manual Driver Loading**
    - You need to explicitly load the database driver.
2. **Connection Management**:
    - Manually acquiring and closing `Connection` and `PreparedStatement` leads to verbose code and potential memory leaks.
3. **SQL Exceptions**:
    - Returns high-level exceptions (`SQLException`) without detailed error granularity (e.g., duplicate key violations, timeouts).
4. **Resource Handling**:
    - Resources like `Connection`, `Statement` need manual closing in a `finally` block.
5. **No Built-in Connection Pooling**:
    - You need to implement database connection pooling manually.

---

### **Spring Boot JDBC Simplifications**
- **Dependencies**:
    - Add the following dependencies in `pom.xml`:
      ```xml
      <!-- JDBC Template -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-jdbc</artifactId>
      </dependency>
      <!-- H2 Database -->
      <dependency>
          <groupId>com.h2database</groupId>
          <artifactId>h2</artifactId>
          <scope>runtime</scope>
      </dependency>
      ```
    - For MySQL or PostgreSQL, replace with appropriate database drivers.

#### **Features of Spring Boot JDBC**:
1. **JDBC Template**:
    - Simplifies database interactions by removing verbose code.
    - Auto-configures the database connection (`DataSource` bean).

2. **Exception Handling**:
    - Maps general `SQLException` to specific exceptions (e.g., `DuplicateKeyException`, `QueryTimeoutException`).

3. **Connection Management**:
    - Automatically manages `Connection`, `Statement`, and `ResultSet` lifecycles.
    - Closes resources automatically.

4. **Integrated Connection Pooling**:
    - **HikariCP** connection pooling is integrated by default.
    - Spring Boot uses HikariCP’s efficient and lightweight connection pool.

#### **Required Configuration (application.properties)**:
- **Minimal Setup**:
    ```properties
    spring.datasource.url=jdbc:h2:mem:userdb
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=sa
    spring.datasource.password= 
    ```

- **Optional Connection Pool Configuration**:
    ```properties
    spring.datasource.hikari.maximum-pool-size=10
    spring.datasource.hikari.minimum-idle=5
    ```

---

#### **Spring JDBC Example**
##### 1. Plain JDBC Connection with Spring
Steps:
1. Create a `Pojo Class`: Represents database entities (e.g., `User` with fields like `userId`, `username`, `age`).
2. Create a **Repository Class** (`@Repository`) for database interactions.
    - Autowire `JdbcTemplate`.
3. Define a **Service Layer** for business logic (`@Service`).

Example:
- **Repository**:
    ```java
    @Repository
    public class UserRepository {

        @Autowired
        private JdbcTemplate jdbcTemplate;

        public void createTable() {
            jdbcTemplate.execute("CREATE TABLE users (userId INT AUTO_INCREMENT, username VARCHAR(255), age INT)");
        }

        public void insertUser(String username, int age) {
            jdbcTemplate.update("INSERT INTO users (username, age) VALUES (?, ?)", username, age);
        }

        public List<User> getUsers() {
            return jdbcTemplate.query("SELECT * FROM users", new RowMapper<User>() {
                @Override
                public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                    User user = new User();
                    user.setUserId(rs.getInt("userId"));
                    user.setUsername(rs.getString("username"));
                    user.setAge(rs.getInt("age"));
                    return user;
                }
            });
        }
    }
    ```

- **Service**:
    ```java
    @Service
    public class UserService {

        @Autowired
        private UserRepository userRepository;

        public void insertAndFetchUsers() {
            userRepository.createTable();
            userRepository.insertUser("John Doe", 30);
            List<User> users = userRepository.getUsers();
            users.forEach(System.out::println);
        }
    }
    ```

---

### **Spring Boot’s JDBC Templates**
#### **Frequent Utility Methods**
1. **For Write Operations (Insert/Update/Delete)**:
    - Using `update(...)`:
      ```java
      jdbcTemplate.update("INSERT INTO users (username, age) VALUES (?, ?)", username, age);
      ```

2. **For Read Operations**:
    - Using `query(...)`:
      ```java
      jdbcTemplate.query("SELECT * FROM users", new RowMapper<User>() {
          @Override
          public User mapRow(ResultSet rs, int rowNum) throws SQLException {
              ...
          }
      });
      ```

    - Using `queryForObject(...)`:
      ```java
      String sql = "SELECT COUNT(*) FROM users";
      Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
      ```

    - Using `queryForList(...)`:
      ```java
      List<String> usernames = jdbcTemplate.queryForList("SELECT username FROM users", String.class);
      ```

---

### **Spring JDBC Advantages**
1. Simplifies boilerplate JDBC code.
2. Built-in support for connection lifecycle management.
3. Provides default exception translation (granular exceptions).
4. Built-in connection pool (HikariCP).
5. Easily configurable via `application.properties`.

---

### **Spring Boot Flow Recap**
1. Application starts, `DataSource` and `JdbcTemplate` are auto-configured.
2. Based on DB driver and connection properties, a connection pool is created.
3. Devs use `JdbcTemplate` for CRUD operations without worrying about drivers, connections, or resource handling.

---

This concludes Spring JDBC Part 1. Up next: **JPA with Hibernate** (ORM Framework). Let me know if you have further questions!