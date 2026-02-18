# Proxy Design Pattern - Detailed Notes

## Overview
- **Category:** Structural Design Pattern
- **Definition:** Provides a representative or placeholder for another object to control access to it

---

## Core Concept

### How Proxy Works

```
┌──────────┐         ┌──────────┐         ┌─────────────┐
│  Client  │ ──────► │  Proxy   │ ──────► │ Real Object │
└──────────┘         └──────────┘         └─────────────┘
                           │
                     Controls Access
```

**Key Points:**
- Client **never** directly talks to the Real Object
- All communication happens **through the Proxy**
- Proxy controls and manages access to the Real Object

---

## Use Cases of Proxy Design Pattern

### 1. Access Control

| Aspect | Description |
|--------|-------------|
| **Purpose** | Restricts access to sensitive operations based on user permissions |
| **Example** | Before allowing a delete operation, proxy checks if user has permission |
| **Flow** | Client → Proxy (checks permission) → Real Object (if allowed) |

```
Client wants DELETE → Proxy checks permission → If allowed → Execute DELETE
                                              → If denied → Block request
```

### 2. Pre and Post Processing

**Real-World Example: Database Transactions in Spring Boot**

```
┌─────────────────────────────────────────────────────────┐
│                    @Transactional                        │
├─────────────────────────────────────────────────────────┤
│  PRE-PROCESSING:   Start Transaction                    │
│  ─────────────────────────────────────────────────────  │
│  ACTUAL TASK:      Perform DB Operation                 │
│  ─────────────────────────────────────────────────────  │
│  POST-PROCESSING:  If success → Commit Transaction      │
│                    If failure → Rollback Transaction    │
└─────────────────────────────────────────────────────────┘
```

**How it works internally:**
1. When you annotate with `@Transactional`, proxy intercepts the call
2. **Pre-processing:** Proxy starts the transaction
3. **Actual Task:** DB operation is performed
4. **Post-processing:** Proxy commits or rollbacks based on success/failure

---

## Class Diagram

```
                    ┌─────────────────────┐
                    │    <<interface>>    │
                    │       Subject       │
                    │─────────────────────│
                    │  + operation()      │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │   RealSubject   │              │  ProxySubject   │
    │   (Concrete)    │◄─────────────│   (Concrete)    │
    │─────────────────│   has-a      │─────────────────│
    │ + operation()   │  reference   │ - realSubject   │
    │   (actual impl) │              │ + operation()   │
    └─────────────────┘              └─────────────────┘
```

### Key Relationships

| Relationship | Description |
|--------------|-------------|
| **Interface** | Both Real Subject and Proxy implement the same interface |
| **Reference** | Proxy holds a reference to Real Subject |
| **Delegation** | Proxy delegates actual work to Real Subject after its checks |

---

## Code Implementation

### Step 1: Define the Interface

```java
// Subject Interface
interface EmployeeDao {
    void getEmployeeInfo();
    void createEmployee();
}
```

### Step 2: Create Real Implementation

```java
// Real Subject - Actual Implementation
class EmployeeDaoImpl implements EmployeeDao {
    
    @Override
    public void getEmployeeInfo() {
        // Fetching data from DB
        System.out.println("Fetching employee info from database...");
    }
    
    @Override
    public void createEmployee() {
        // Creating employee in DB
        System.out.println("Creating new employee in database...");
    }
}
```

### Step 3: Create Proxy Class

```java
// Proxy Subject - Controls Access
class EmployeeDaoProxy implements EmployeeDao {
    
    // Reference to real object
    private EmployeeDaoImpl employeeDaoImpl;
    
    // Fields for validation
    private String clientRole;
    
    // Constructor
    public EmployeeDaoProxy(String clientRole) {
        this.clientRole = clientRole;
        // Creating reference to actual object
        this.employeeDaoImpl = new EmployeeDaoImpl();
    }
    
    @Override
    public void getEmployeeInfo() {
        // Access Control: Admin OR User can access
        if (clientRole.equals("ADMIN") || clientRole.equals("USER")) {
            // Forward request to actual object
            employeeDaoImpl.getEmployeeInfo();
        } else {
            throw new RuntimeException("Access Denied!");
        }
    }
    
    @Override
    public void createEmployee() {
        // Access Control: Only Admin can create
        if (clientRole.equals("ADMIN")) {
            // Forward request to actual object
            employeeDaoImpl.createEmployee();
        } else {
            throw new RuntimeException("Access Denied!");
        }
    }
}
```

### Step 4: Client Usage

```java
// Client Code
public class Client {
    public static void main(String[] args) {
        
        // Creating proxy object with USER role
        EmployeeDao proxy = new EmployeeDaoProxy("USER");
        
        // ✅ This will work - Users can get employee info
        proxy.getEmployeeInfo();
        // Output: Fetching employee info from database...
        
        // ❌ This will throw exception - Only Admins can create
        proxy.createEmployee();
        // Output: RuntimeException: Access Denied!
    }
}
```

---

## Access Control Matrix (Example)

| Operation | ADMIN | USER | GUEST |
|-----------|-------|------|-------|
| `getEmployeeInfo()` | ✅ | ✅ | ❌ |
| `createEmployee()` | ✅ | ❌ | ❌ |

---

## Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        EXECUTION FLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Client creates EmployeeDaoProxy("USER")                     │
│     └── Constructor assigns clientRole = "USER"                 │
│     └── Constructor creates new EmployeeDaoImpl()               │
│                                                                  │
│  2. Client calls proxy.getEmployeeInfo()                        │
│     └── Proxy checks: is role ADMIN or USER? → YES              │
│     └── Proxy forwards to employeeDaoImpl.getEmployeeInfo()     │
│     └── ✅ Success                                               │
│                                                                  │
│  3. Client calls proxy.createEmployee()                         │
│     └── Proxy checks: is role ADMIN? → NO (it's USER)           │
│     └── ❌ Access Denied!                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## What Proxy Does vs Real Object

| Component | Responsibility |
|-----------|----------------|
| **Proxy** | Controls access, validation, pre/post processing |
| **Real Object** | Actual implementation and business logic |

> **Important:** Proxy only does validation and forwards requests. The actual implementation remains in the Real Object.

---

## Real-World Usage in Industry

| Technology | How Proxy is Used |
|------------|-------------------|
| **Spring Boot** | `@Transactional`, AOP (Aspect Oriented Programming) |
| **JUnit** | Mocking frameworks |
| **Java Reflection** | Dynamic proxies |
| **Annotations** | Intercepting method calls |
| **Security** | Authentication and Authorization checks |
| **Caching** | Cache proxy before hitting actual service |
| **Logging** | Log requests before/after actual execution |

---

## Types of Proxies

| Type | Purpose |
|------|---------|
| **Protection Proxy** | Controls access based on permissions |
| **Virtual Proxy** | Lazy initialization of expensive objects |
| **Remote Proxy** | Represents object in different address space |
| **Cache Proxy** | Caches results of expensive operations |
| **Logging Proxy** | Logs method calls and parameters |

---

## Summary

```
┌────────────────────────────────────────────────────────────────┐
│                    PROXY DESIGN PATTERN                         │
├────────────────────────────────────────────────────────────────┤
│  WHAT: A placeholder that controls access to real object       │
│                                                                 │
│  HOW:  Client → Proxy → Real Object                            │
│                                                                 │
│  WHY:  • Access Control (permissions)                          │
│        • Pre/Post Processing (transactions)                    │
│        • Lazy Loading                                          │
│        • Logging, Caching, etc.                                │
├────────────────────────────────────────────────────────────────┤
│  KEY POINTS:                                                    │
│  • Proxy implements same interface as Real Object              │
│  • Proxy holds reference to Real Object                        │
│  • Proxy adds behavior, then delegates to Real Object          │
└────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Proxy sits between** Client and Real Object
2. **Same Interface:** Both Proxy and Real Object implement the same interface
3. **Reference:** Proxy maintains a reference to the Real Object
4. **Control:** Proxy controls access, does validation, pre/post processing
5. **Delegation:** After its work, Proxy delegates to the Real Object
6. **Heavily Used:** In Spring Boot, JUnit, Java Reflection, and many frameworks