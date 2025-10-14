### **Spring Boot JPA | Association Types and Mappings (Part 6: One-to-One Unidirectional and Bidirectional) | Notes**

---

### **Overview**
Understanding entity relationships and mappings in JPA is fundamental for modeling your database effectively in a Spring Boot application. This session focuses on **one-to-one mappings (both unidirectional and bidirectional)**, covering annotations like `@OneToOne`, cascading, fetch types (eager vs lazy), and query behavior.

---

## **1. One-to-One Unidirectional Mapping**

### **Definition**
- **Unidirectional**: One entity (parent) references another entity (child) in **one direction only**, via a foreign key.
- Example: A user (`UserDetail`) can have only one associated address (`UserAddress`). The `UserDetail` table contains a foreign key referencing the `UserAddress` table, but `UserAddress` cannot reference `UserDetail`.

---

### **Setup**
#### Parent Entity (`UserDetail`):
```java
@Entity
@Table(name = "user_detail")
public class UserDetail {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;

    @OneToOne(cascade = CascadeType.ALL) // Cascade is explained below
    @JoinColumn(name = "address_id")    // Define the foreign key column name
    private UserAddress userAddress;
}
```

#### Child Entity (`UserAddress`):
```java
@Entity
@Table(name = "user_address")
public class UserAddress {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String street;
    private String city;
    private String state;
    private String country;
    private String pinCode;
}
```

---

### **Generated Table Schema**
| **Table**       | **Columns**                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `user_detail`    | `id`, `name`, `phone`, `address_id` (foreign key pointing to `user_address.id`) |
| `user_address`   | `id`, `street`, `city`, `state`, `country`, `pinCode`                      |

---

### **Key Behavior**
1. **Foreign Key Management**:
    - Hibernate automatically creates the foreign key (`address_id`) in the parent table (`UserDetail`) pointing to the primary key of the child table (`UserAddress`).

2. **Custom Foreign Key Name**:
    - Using `@JoinColumn`: You can specify the name of the foreign key manually.
   ```java
   @JoinColumn(name = "custom_address_id")
   ```

3. **Composite Keys**:
    - Use `@JoinColumns` when referencing a child entity with a composite key (multiple primary key fields).
   ```java
   @OneToOne
   @JoinColumns({
       @JoinColumn(name = "address_street", referencedColumnName = "street"),
       @JoinColumn(name = "address_pinCode", referencedColumnName = "pinCode")
   })
   ```

---

## **2. Cascade Types**

Cascade types determine whether actions performed on the parent entity should cascade to the associated child entity. **CascadeType** options include:
| **Cascade Type**  | **Description**                                                                          |
|-------------------|------------------------------------------------------------------------------------------|
| `ALL`             | Applies all cascading types (below).                                                    |
| `PERSIST`         | Saves both parent and associated child entity when the parent entity is saved.           |
| `MERGE`           | Updates the child entity when the parent entity is updated.                              |
| `REMOVE`          | Deletes the child entity when the parent entity is removed.                              |
| `REFRESH`         | Synchronizes child entity with DB every time parent entity is refreshed.                 |
| `DETACH`          | Detaches both parent and child entities from the persistence context.                    |

---

### **Cascade Type Example**
#### Insert:
```java
@OneToOne(cascade = CascadeType.PERSIST)
@JoinColumn(name = "address_id")
private UserAddress userAddress;
```

- **Behavior**: When inserting a `UserDetail` entity, the associated `UserAddress` entity is also automatically inserted.

#### Update (Merge):
```java
@OneToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
@JoinColumn(name = "address_id")
private UserAddress userAddress;
```

- **Behavior**: Updates in `UserDetail` will also update changes in `UserAddress`.

#### Delete (Remove):
```java
@OneToOne(cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
@JoinColumn(name = "address_id")
private UserAddress userAddress;
```

- **Behavior**: Deleting a `UserDetail` also removes the associated `UserAddress`.

---

## **3. Fetch Types (Eager vs Lazy Loading)**

#### **Eager Loading**:
- **Definition**: Associated entities are fetched **immediately** along with the parent entity.
- **Default for @OneToOne**:
  ```java
  @OneToOne(fetch = FetchType.EAGER)
  ```

#### **Lazy Loading**:
- **Definition**: Associated entities are fetched **only when explicitly accessed** (improves performance when child entities aren't always required).
- **Controlled Loading**:
  ```java
  @OneToOne(fetch = FetchType.LAZY)
  ```

---

### **Lazy Loading Serialization Issue**
Lazy loading prevents immediate fetching of associated entities, which can cause serialization problems (e.g., Jackson failing while constructing a JSON response).

#### **Example Problem**:
```java
@OneToOne(fetch = FetchType.LAZY)
private UserAddress userAddress;
```
- When a **GET request** is made:
    - Parent fields are fetched, but child fields (`UserAddress`) are lazily-loaded.
    - Serialization can fail due to missing data.
    - **Solution**:
        - Use `@JsonIgnore` to exclude child fields.
        - Alternatively, use DTOs (Data Transfer Objects) and explicitly access child fields.

---

## **4. One-to-One Bidirectional Mapping**

### **Definition**
- Bidirectional association allows navigation from the **parent entity to child entity** and **child entity to parent entity**.
- **DB Structure DOES NOT change**. Foreign key still exists in the parent entity's table only.

---

### **Setup**
#### Parent Entity (`UserDetail`):
```java
@Entity
@Table(name = "user_detail")
public class UserDetail {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String phone;

    @OneToOne(cascade = CascadeType.ALL, mappedBy = "userDetail") // Bidirectional mapping
    private UserAddress userAddress; // Inverse side
}
```

#### Child Entity (`UserAddress`):
```java
@Entity
@Table(name = "user_address")
public class UserAddress {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String street;
    private String city;
    private String state;
    private String country;
    private String pinCode;

    @OneToOne
    @JoinColumn(name = "user_detail_id") // Foreign key still exists in parent table
    private UserDetail userDetail;      // Owner side
}
```

---

### **Key Concepts**
1. **Owning Side (`mappedBy`)**:
    - `mappedBy` defines the inverse side of the relationship.
    - For `@OneToOne`, the **owning side holds the foreign key**.

2. **DB Schema**:
    - `user_detail`: Contains the foreign key `address_id`.
    - `user_address`: No foreign key columns are created due to `mappedBy`.

3. **Query Behavior**:
    - Accessing the parent (`UserDetail`) allows backward traversal to the child (`UserAddress`) without additional DB columns.

---

### **Infinite Recursion Problem**
When bidirectional associations are serialized, **Jackson** enters an infinite recursion loop (parent referencing the child, and child referencing the parent).

#### **Solution: Suppress Recursive Serialization**
1. **`@JsonManagedReference` and `@JsonBackReference`**:
    - `@JsonManagedReference`: Use on the **owning side** (parent).
    - `@JsonBackReference`: Use on the **inverse side** (child).
   ```java
   @JsonManagedReference
   private UserAddress userAddress;

   @JsonBackReference
   private UserDetail userDetail;
   ```

2. **`@JsonIdentityInfo`**:
    - Assigns a unique identifier to objects during serialization, preventing infinite recursion.
   ```java
   @JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
   @Entity
   public class UserDetail { ... }
   ```

---

## **5. Summary**
| **Mapping Type**      | **Primary Use Case**                                   |
|-----------------------|-------------------------------------------------------|
| **One-to-One Unidirectional** | Associate one entity (parent) with one child entity in one direction only.                    |
| **One-to-One Bidirectional** | Allows navigation between both entities without affecting DB schema.                          |

---

## **Next Steps**
1. Mapping **One-to-Many** and **Many-to-Many** relationships.
2. Writing **custom JPQL queries**, criteria queries, and implementing pagination and sorting.

Let me know if you'd like more clarification on any specific sections! 😊