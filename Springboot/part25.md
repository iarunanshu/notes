Here's a summarized set of notes based on your transcript for **JPA Entity Mapping and Annotations**:

---

### **Spring Boot JPA | Entity Mapping Annotations and Schema Management**

---

### **1. Hibernate Schema Management: `spring.jpa.hibernate.ddl-auto`**
This property in `application.properties` tells Hibernate how to handle schema creation and updates during application startup.

#### **Supported Values:**
1. **none**:
    - Hibernate does not create, update, or drop the schema.
    - **Production Recommendation**: Always use `none` in production systems. Schema changes should be handled via scripts/external SQL files.

2. **validate**:
    - Validates the schema structure against JPA entities.
    - Ensures DB schema matches the entity structure. Throws exceptions if schema differs.
    - Does not create, update, or drop the schema.

3. **update**:
    - Creates or updates the schema if it does not match the entity structure.
    - Does **NOT** delete or drop tables/columns.
    - Recommended for development environments.

4. **create**:
    - Drops the existing schema and creates a new schema at startup.
    - Deletes all existing data in the database.

5. **create-drop**:
    - Similar to `create`, but also **drops** the schema during application shutdown.

---

### **2. Difference Between Database and Schema**
1. **Database**:
    - A container for tables, views, and schemas.
    - One database can serve multiple applications.

2. **Schema**:
    - A **logical grouping** of tables within a database.
    - Useful for organizing tables for different teams or modules.

#### **Example**:
- **Database**: `company_db`
- Schemas:
    - `onboarding`
    - `sales`
    - `hr`
- Tables in schemas:
    - `onboarding.users`, `sales.orders`, `hr.employees`

Schemas allow better management of permissions and logical boundaries.

---

### **3. Entity Table Mapping**
#### **Annotations**:
1. **`@Entity`**:
    - Marks a class as a JPA-managed entity.
    - Equivalent to a table in the database.
    - Each instance of the entity corresponds to a row in the table.

2. **`@Table`**:
    - Maps the entity to a specific table and schema in the database.
    - Optional. If not provided, the table name defaults to the entity name.

#### **Attributes of `@Table`:**
1. **`name`**:
    - Explicitly set the table name.
    - Default is the camel-cased entity name converted to snake_case.

2. **`schema`**:
    - Specifies the schema to which the table belongs.
    - Example:
      ```java
      @Table(name = "user_detail", schema = "onboarding")
      ```

3. **`uniqueConstraints`**:
    - Defines uniqueness constraints at the table level.
    - Example (Composite Unique Constraint):
      ```java
      @Table(uniqueConstraints = @UniqueConstraint(columnNames = {"email", "username"}))
      ```

4. **`indexes`**:
    - Create table indexes for specified columns.
    - Example:
      ```java
      @Table(indexes = {@Index(name = "phone_index", columnList = "phone")})
      ```

---

### **4. Column-Level Mapping with `@Column`**
#### **Attributes of `@Column`**:
1. **`name`**:
    - Column name in the database.
    - Default: Field name converted to snake_case.

2. **`unique`**:
    - Marks the column as unique.
    - Example:
      ```java
      @Column(name = "email_address", unique = true)
      ```

3. **`nullable`**:
    - Allows/disallows null values in the column.

4. **`length`**:
    - Specifies maximum column length (used for `VARCHAR` columns).
    - Default: 255 characters.

---

### **5. Primary Key Annotations**
#### **`@Id`**:
- Used to mark a field as the primary key of the table.
- The primary key uniquely identifies each row in the table.
- An entity can only have **one primary key**.

#### **Composite Primary Key**:
- When a table has a **composite key** (more than one column as primary key), you cannot directly annotate multiple fields with `@Id`. Use one of the following approaches:
    1. **`@IdClass`**:
        - Example:
          ```java
          @IdClass(UserCompositeKey.class)
          public class UserDetail {
              @Id private String name;
              @Id private String address;
          }
          public class UserCompositeKey implements Serializable {
              private String name;
              private String address;
              // Implement equals() and hashCode()
          }
          ```

    2. **`@EmbeddedId`** and **`@Embeddable`**:
        - Example:
          ```java
          @Embeddable
          public class UserKey implements Serializable {
              private String name;
              private String address;
              // Implement equals() and hashCode()
          }
   
          @Entity
          public class UserDetail {
              @EmbeddedId
              private UserKey userKey;
          }
          ```

---

### **6. Auto-Generated Primary Keys**
Use `@GeneratedValue` for automatically generating primary key values.

#### **Strategies**:
1. **`GenerationType.IDENTITY`**:
    - Relies on the database’s AUTO_INCREMENT/identity column feature.
    - Example:
      ```java
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      ```

2. **`GenerationType.SEQUENCE`**:
    - Uses database sequences for key generation.
    - Preferred in production and **more efficient than `IDENTITY`**.

   Example:
   ```java
   @Id
   @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq_gen")
   @SequenceGenerator(name = "user_seq_gen", sequenceName = "user_sequence", initialValue = 100, allocationSize = 5)
   private Long id;
   ```

   **Explanation**:
    - Hibernate caches IDs to reduce DB hits (based on `allocationSize`).
    - Generates unique IDs for multiple entities using the same sequence.

3. **`GenerationType.TABLE`**:
    - Uses a separate database table to maintain and generate IDs.
    - Least efficient compared to `SEQUENCE` and `IDENTITY`.

---

### **7. Key Differences Between `IDENTITY` and `SEQUENCE`**
| **Feature**          | **IDENTITY**               | **SEQUENCE**               |
|----------------------|---------------------------|---------------------------|
| **Behavior**         | DB handles ID generation. | Hibernate controls the sequence. |
| **Efficiency**       | No caching. Every insert requires DB interaction. | IDs can be cached for efficient batch inserts. |
| **Portability**      | DB-specific (varies across systems). | Cross-DB compatible (standardized). |
| **Concurrency**      | Handles auto-increments natively. | Better concurrent handling via Hibernate. |

---

### **8. Practical Example**
#### **Creating a User Table with Primary Key, Unique Field, and Cacheable Sequence Strategy**
```java
@Entity
@Table(name = "users", schema = "onboarding", 
       uniqueConstraints = @UniqueConstraint(columnNames = {"email"}))
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq_gen")
    @SequenceGenerator(name = "user_seq_gen", sequenceName = "user_seq", initialValue = 100, allocationSize = 10)
    private Long id;

    @Column(name = "email", nullable = false, unique = true, length = 50)
    private String email;

    @Column(name = "name", length = 100)
    private String name;
}
```

#### **Generated Schema**
SQL Schema auto-generated by Hibernate:
```sql
CREATE SCHEMA onboarding;
CREATE SEQUENCE onboarding.user_seq START WITH 100 INCREMENT BY 10;

CREATE TABLE onboarding.users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100)
);
```

---

### **9. Best Practices**
1. **Always Use `NONE` in Production**:
    - Schema management should be handled explicitly via scripts.
2. **Use Sequences for Primary Keys**:
    - Efficient in concurrent environments and provides better portability.
3. **Test Composite Keys if Necessary**:
    - Use `@IdClass` or `@EmbeddedId` for structured composite keys.

---

### **Coming Next**
1. Exploring Relationships:
    - **One-To-One, One-To-Many, Many-To-Many Mapping**.
2. Writing Custom Queries:
    - **JPQL**, **criteria queries**, pagination, and sorting.

Let me know if you'd like to dive deeper into any specific concept! 😊