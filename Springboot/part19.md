# Spring Boot Response Handling & HTTP Status Codes

## Response Components

A response in Spring Boot consists of three parts:

1. **Status Code** - HTTP response code (200, 500, etc.)
2. **Header** - Additional information sent in response
3. **Body** - Actual data to be sent in response

## ResponseEntity in Spring Boot

### Basic Structure

```java
ResponseEntity<T>
```
- Generic class where `T` represents the type of body
- `T` determines what type of data can be sent in the body

### Example Implementation

```java
@RestController
public class UserController {
    
    @GetMapping("/getUser")
    public ResponseEntity<String> getUser() {
        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("Custom-Header", "HeaderValue");
        
        // Build response with status, headers, and body
        return ResponseEntity
            .status(HttpStatus.OK)
            .headers(headers)
            .body("My response body");
    }
}
```

### Builder Pattern in ResponseEntity

**Important**: Body must always be last in the chain

```java
// Correct ✅
ResponseEntity.status(HttpStatus.OK)
    .headers(headers)
    .body("response");  // body() returns ResponseEntity<T>

// Incorrect ❌
ResponseEntity.status(HttpStatus.OK)
    .body("response")
    .headers(headers);  // Won't compile - body() is terminal
```

**Method Return Types:**
- `status()` → returns `BodyBuilder`
- `headers()` → returns `BodyBuilder`
- `body()` → returns `ResponseEntity<T>` (final)

### Handling Empty Body

When no body is needed (e.g., DELETE operations):

```java
@DeleteMapping("/deleteUser")
public ResponseEntity<Void> deleteUser() {
    return ResponseEntity
        .status(HttpStatus.NO_CONTENT)
        .headers(headers)
        .build();  // build() internally calls body(null)
}
```

## Direct Object Return vs ResponseEntity

### Without ResponseEntity

```java
@RestController
public class UserController {
    
    @GetMapping("/getUser")
    public User getUser() {
        return new User();  // Spring wraps it in ResponseEntity internally
        // Default status: 200 OK
    }
}
```

### When to Use ResponseEntity

- When you need to set specific status codes
- When you need to add custom headers
- When you need fine control over the response

## @ResponseBody Annotation

### Purpose
Tells Spring to consider the return value as response body, not as a view name

### Where It's Required

#### With @RestController (Not Required)
```java
@RestController  // Contains @ResponseBody internally
public class UserController {
    @GetMapping("/getUser")
    public String getUser() {
        return "user data";  // Treated as response body
    }
}
```

#### With @Controller (Required)
```java
@Controller
public class UserController {
    @GetMapping("/getUser")
    @ResponseBody  // Required here
    public String getUser() {
        return "user data";  // Without @ResponseBody, Spring looks for a view
    }
}
```

**Without @ResponseBody in @Controller:**
- Spring treats return value as view name
- Looks for JSP/HTML file with that name
- Throws "Not Found" error if view doesn't exist

## HTTP Status Code Categories

### 1XX - Informational
- Interim response codes
- Not final responses
- Rarely used in practice

### 2XX - Success
Request successfully processed

### 3XX - Redirection
Client needs to take additional action

### 4XX - Client Error
Validation errors, client fault

### 5XX - Server Error
Server-side failures

## Detailed Status Codes

### 2XX - Success Codes

#### 200 OK
```java
// GET method - with response body
@GetMapping("/user")
public ResponseEntity<User> getUser() {
    return ResponseEntity.ok(user);
}

// POST method - idempotent calls
@PostMapping("/user")
public ResponseEntity<User> createUser(@RequestBody User user) {
    // If user already exists (idempotent)
    return ResponseEntity.ok(existingUser);
}
```

#### 201 Created
```java
// POST - New resource created
@PostMapping("/user")
public ResponseEntity<User> createUser(@RequestBody User user) {
    User created = userService.create(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

#### 202 Accepted
```java
// Async processing - Export/Import operations
@PostMapping("/export")
public ResponseEntity<Void> exportData() {
    // Spawn async thread for processing
    asyncService.startExport();
    return ResponseEntity.status(HttpStatus.ACCEPTED).build();
}
```

#### 204 No Content
```java
// DELETE - Success with no response body
@DeleteMapping("/user/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
}
```

#### 206 Partial Content
```java
// Bulk operations with partial success
@PostMapping("/users/bulk")
public ResponseEntity<BulkResult> bulkCreate(@RequestBody List<User> users) {
    // 95 success, 5 failed out of 100
    BulkResult result = userService.bulkCreate(users);
    return ResponseEntity.status(HttpStatus.PARTIAL_CONTENT).body(result);
}
```

### 3XX - Redirection Codes

#### 301 Moved Permanently
```java
@GetMapping("/old/getUser")
public ResponseEntity<Void> oldApi() {
    HttpHeaders headers = new HttpHeaders();
    headers.add("Location", "/api/new/getUser");
    
    return ResponseEntity
        .status(HttpStatus.MOVED_PERMANENTLY)
        .headers(headers)
        .build();
}

@GetMapping("/api/new/getUser")
public ResponseEntity<String> newApi() {
    return ResponseEntity.ok("Success");
}
```

**Client Behavior:**
- Clients automatically follow redirects
- Can be disabled in client settings
- Example: Postman's "Automatically follow redirects"

#### 308 Permanent Redirect
- Similar to 301 but **HTTP method cannot change**
- POST → must redirect to POST
- GET → must redirect to GET

#### 304 Not Modified
```java
@GetMapping("/resource")
public ResponseEntity<Resource> getResource(
        @RequestHeader(value = "If-Modified-Since", required = false) 
        String ifModifiedSince) {
    
    // Check if resource modified since client's last fetch
    if (!isModified(ifModifiedSince)) {
        return ResponseEntity.status(HttpStatus.NOT_MODIFIED).build();
    }
    
    // Return updated resource
    return ResponseEntity.ok()
        .header("Last-Modified", getLastModified())
        .body(resource);
}
```

**Common Misuse:**
- ❌ Using with PATCH for unchanged data
- ✅ Only for client caching scenarios with GET

### 4XX - Client Error Codes

#### 400 Bad Request
```java
// Missing required fields
@PostMapping("/user")
public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
    // Validation fails - returns 400 automatically
    return ResponseEntity.ok(userService.create(user));
}
```

#### 401 Unauthorized
```java
// Missing authentication
@GetMapping("/secure/data")
public ResponseEntity<Data> getSecureData(
        @RequestHeader("Authorization") String token) {
    if (!isValidToken(token)) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
    return ResponseEntity.ok(data);
}
```

#### 403 Forbidden
```java
// Insufficient permissions
@PutMapping("/admin/settings")
public ResponseEntity<Settings> updateSettings(Principal principal) {
    if (!isAdmin(principal)) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
    }
    return ResponseEntity.ok(settings);
}
```

#### 404 Not Found
```java
// Resource doesn't exist
@GetMapping("/user/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    Optional<User> user = userService.findById(id);
    return user.map(ResponseEntity::ok)
               .orElse(ResponseEntity.notFound().build());
}
```

#### 405 Method Not Allowed
```java
// Wrong HTTP method for endpoint
// If endpoint is @GetMapping but client sends POST
```

#### 409 Conflict
```java
// Resource already being processed
@PatchMapping("/user/{id}")
public ResponseEntity<User> updateUser(@PathVariable Long id) {
    if (isLocked(id)) {
        return ResponseEntity.status(HttpStatus.CONFLICT).build();
    }
    
    // Lock resource with TTL
    lockResource(id, 10); // 10 seconds TTL
    
    try {
        User updated = userService.update(id);
        return ResponseEntity.ok(updated);
    } finally {
        unlockResource(id);
    }
}
```

#### 422 Unprocessable Entity
```java
// Business logic failure
@PostMapping("/user")
public ResponseEntity<User> createUser(@RequestBody User user) {
    // Business rule: No France users allowed
    if ("France".equals(user.getCountry())) {
        return ResponseEntity
            .status(HttpStatus.UNPROCESSABLE_ENTITY)
            .body("France users not allowed");
    }
    return ResponseEntity.ok(userService.create(user));
}
```

#### 429 Too Many Requests
```java
// Rate limiting
@GetMapping("/api/data")
public ResponseEntity<Data> getData(String userId) {
    if (rateLimiter.exceedsLimit(userId)) {
        return ResponseEntity
            .status(HttpStatus.TOO_MANY_REQUESTS)
            .header("Retry-After", "60")
            .build();
    }
    return ResponseEntity.ok(data);
}
```

### 5XX - Server Error Codes

#### 500 Internal Server Error
```java
// Generic server error
try {
    return ResponseEntity.ok(processRequest());
} catch (Exception e) {
    // NullPointerException, DB down, etc.
    return ResponseEntity
        .status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body("Internal server error");
}
```

#### 501 Not Implemented
```java
// API under development
@PostMapping("/future-feature")
public ResponseEntity<String> futureFeature() {
    return ResponseEntity
        .status(HttpStatus.NOT_IMPLEMENTED)
        .body("Feature coming soon");
}
```

#### 502 Bad Gateway
- Server acting as proxy gets invalid response
- Example: Nginx can't reach application
- Port misconfiguration or timeout issues

### 1XX - Informational Codes

#### 100 Continue
```java
// Client checks if server can handle request
// Step 1: Client sends headers with "Expect: 100-continue"
// Step 2: Server validates and responds with 100
// Step 3: Client sends actual request
// Step 4: Server processes and sends final response
```

## Best Practices

### 1. Choose Appropriate Status Codes
```java
// Good ✅
@DeleteMapping("/user/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build(); // 204
}

// Bad ❌
@DeleteMapping("/user/{id}")
public ResponseEntity<String> delete(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.ok("Deleted"); // 200 with unnecessary body
}
```

### 2. Consistent Error Handling
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Error> handleNotFound(ResourceNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new Error(e.getMessage()));
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<Error> handleValidation(ValidationException e) {
        return ResponseEntity.badRequest()
            .body(new Error(e.getMessage()));
    }
}
```

### 3. Use ResponseEntity for Control
```java
// When you need specific status/headers
public ResponseEntity<User> createUser(User user) {
    User created = userService.create(user);
    
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("X-User-Id", created.getId())
        .body(created);
}
```

## Common Interview Questions

### Q1: Difference between 200 and 201?
- **200**: General success, used for GET and idempotent POST
- **201**: Specifically for resource creation (new row in DB)

### Q2: When to use 304?
- Only with GET requests
- Client caching scenarios
- NOT for PATCH operations with unchanged data

### Q3: Difference between 401 and 403?
- **401**: Not authenticated (no valid credentials)
- **403**: Authenticated but not authorized (no permission)

### Q4: What is idempotent in context of HTTP?
- Multiple identical requests produce same result
- GET, PUT, DELETE are idempotent
- POST is typically not idempotent

## Key Takeaways

1. **ResponseEntity** provides full control over HTTP response
2. **Builder pattern** - body() must be last
3. **@ResponseBody** needed with @Controller, not @RestController
4. **Choose correct status codes** - impacts client behavior
5. **5XX errors should be minimized** - indicates server problems
6. **3XX codes** useful for API migration
7. **Don't misuse status codes** - e.g., 304 for PATCH operations