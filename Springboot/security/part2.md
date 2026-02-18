# Spring Boot Security Architecture - Notes

## Overview
- Authentication and Authorization are essential to protect resources from attacks
- Spring Security is built on the Filter Chain architecture

---

## Key Concepts

### Authentication vs Authorization

| Authentication | Authorization |
|---|---|
| Verify **who you are** | Check **what you are allowed to do** |
| Username/Password verification | Permission/Role-based access control |
| User identity confirmation | Resource access restrictions |

**Example:** You may be authenticated to access a system, but only authorized to view data (not update/delete)

---

## Spring Security Architecture

### 1. Request Flow in Filter Chain

**Standard Flow (without Security):**
```
Request → Servlet Container → Filter Chain (Filter 1, 2, ...n) 
       → Dispatcher Servlet → Interceptor → Controller (Business Logic)
```

**With Spring Security:**
```
Request → Servlet Container → Filter Chain (Filter 1, 2, ...Security Filter Chain, ...n)
       → Dispatcher Servlet → Interceptor → Controller
```

**Key Point:** Security Filter Chain is inserted as one of the filters in the existing filter chain, not a separate entity.

### 2. Authentication Method-Specific Filters

Different authentication methods use different filters:

| Authentication Method | Applicable Filters |
|---|---|
| Form-Based (Session) | Specific filters for form login |
| Basic Authentication | Specific filters for basic auth |
| JWT | Specific filters for JWT |
| OAuth | Specific filters for OAuth |

**Important:** Not all filters execute for every request. Only filters relevant to the chosen authentication method are invoked.

---

## Core Components

### 1. **Security Filter Chain**
- Consists of multiple security filters
- Each filter handles specific aspects of authentication
- Only relevant filters execute based on authentication method

### 2. **Authentication Object**
Contains:
- `isAuthenticated`: Boolean (true/false)
- `roles`: User roles/authorities
- Other authentication details

**Flow:**
- **Initial State:** Incomplete, `isAuthenticated = false`
- **After Provider Processing:** Complete, `isAuthenticated = true/false`

### 3. **Authentication Manager (Interface)**
- **Implementation:** ProviderManager (default)
- **Role:** Bridge between filter and authentication provider
- **Responsibility:** Delegates authentication request to appropriate provider based on authentication method

### 4. **Authentication Providers**
Perform actual authentication work. Examples:

#### **DAO Authentication Provider (Username/Password)**
- Handles stateful/session-based authentication
- **Process:**
    1. Receives raw password from user
    2. Hashes the password using Password Encoder
    3. Fetches stored user details from User Detail Service
    4. Compares hashed passwords
    5. Returns authenticated or failed Authentication object

#### **Other Providers**
- JWT Authentication Provider (for stateless JWT)
- OAuth Authentication Provider (for OAuth flows)

### 5. **Password Encoder**
- Hashes passwords during registration
- Hashes incoming password during validation
- Ensures secure password comparison (never stores raw passwords)

### 6. **User Detail Service (Interface)**
Loads user information from storage:

| Implementation | Storage |
|---|---|
| In-Memory | Application memory (temporary) |
| JDBC User Detail Manager | Database (persistent) |

### 7. **Security Context**
- **Role:** Stores authenticated user information
- **Timing:** Created after successful authentication
- **Availability:** Added to request, accessible in controller
- **Content:** Complete Authentication object with user details and roles

---

## Complete Request Processing Flow

```
1. Request arrives at filter chain
   ↓
2. One of the Security Filters is invoked (based on authentication method)
   ↓
3. Filter creates incomplete Authentication object
   (isAuthenticated = false)
   ↓
4. Filter passes to Authentication Manager
   ↓
5. Authentication Manager (Provider Manager) delegates to appropriate provider
   (e.g., DAO Authentication Provider for username/password)
   ↓
6. Provider processes authentication:
   - Encode user password
   - Load existing user details from User Detail Service
   - Compare passwords
   ↓
7. Provider returns complete Authentication object
   (isAuthenticated = true/false)
   ↓
8. Authentication Manager returns to Filter
   ↓
9. Filter checks result:
   - If authenticated → Store in Security Context
   - If not authenticated → Throw "Bad Credentials" exception
   ↓
10. Request with Security Context continues through:
    - Dispatcher Servlet
    - Interceptors
    - Controller
```

---

## Architecture as a Template

**Important Concept:**
- This architecture is a **base template** for all authentication methods
- Each authentication method follows this template with variations:
    - **Different filters** are invoked
    - **Different providers** are used
    - **Different steps** are added/skipped
    - **Different data** is stored in Security Context

**Example Variations:**
- Form-based login: Full flow with session management
- Basic Authentication: Skips certain steps
- JWT: Stateless processing (no session management)

---

## Dependencies Required

### Mandatory
```
spring-boot-starter-security
```
Includes:
- Filter chain setup
- Authentication manager
- All security providers
- Password encoders

### Optional
```
spring-session-jdbc
```
Only required if:
- Using session-based authentication
- Storing sessions in database
- Need persistent session management

**Note:** For new Spring Boot projects, use Spring Initializer and select "Spring Security" alongside "Spring Web"

---

## Key Takeaways

1. ✅ Spring Security is integrated into the existing Filter Chain
2. ✅ Only relevant filters execute based on authentication method
3. ✅ Authentication Manager acts as a router to appropriate providers
4. ✅ Providers do actual authentication work (password encoding, user lookup)
5. ✅ Security Context stores authenticated user information
6. ✅ Architecture is a template; variations exist for each authentication method
7. ✅ Passwords are never stored/compared in raw form (always encoded)
8. ✅ Different storage options exist (in-memory or database) for user details