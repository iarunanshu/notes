# Spring Boot Annotations - Complete Notes

## Overview
These annotations help with **Handler Mapping** - the process where DispatcherServlet identifies which controller and method should handle incoming HTTP requests.

## Controller Annotations

### @Controller
- **Purpose**: Indicates class is responsible for handling HTTP requests
- Spring Boot considers this class eligible to handle HTTP requests
- Gets added to the list of controller classes
- **Important**: Without @ResponseBody, return values are treated as view names (JSP, HTML)

### @RestController
- **Purpose**: Combination of @Controller + @ResponseBody
- All methods automatically return HTTP responses (not view names)
- More convenient for REST APIs - no need to add @ResponseBody to each method
```java
@RestController = @Controller + @ResponseBody
```

### @ResponseBody
- **Purpose**: Tells Spring that method return type should be HTTP response body
- Without it: Spring treats return value as view name (e.g., "hello" → looks for hello.jsp)
- With it: Return value sent as-is as REST response

**Example - Controller vs RestController:**
```java
// Using @Controller - needs @ResponseBody
@Controller
public class SampleController {
    @GetMapping("/api/fetchUser")
    @ResponseBody  // Required for REST response
    public String getUserDetails() {
        return "User data";
    }
}

// Using @RestController - @ResponseBody implicit
@RestController
public class SampleController {
    @GetMapping("/api/fetchUser")
    // No @ResponseBody needed
    public String getUserDetails() {
        return "User data";
    }
}
```

## Request Mapping Annotations

### @RequestMapping
- **Purpose**: Maps HTTP requests to controller methods
- Can be used at class level (common path) and method level
- **Parameters**:
    - `path` or `value`: URL path (both are aliases)
    - `method`: HTTP method (GET, POST, PUT, DELETE)

**Example:**
```java
@RestController
@RequestMapping("/api")  // Class level - common path
public class SampleController {
    
    @RequestMapping(path = "/fetchUser", method = RequestMethod.GET)
    public String getUser() {
        return "User details";
    }
}
```

### HTTP Method Specific Mappings
More readable alternatives to @RequestMapping:

- **@GetMapping**: For GET requests
- **@PostMapping**: For POST requests
- **@PutMapping**: For PUT requests
- **@DeleteMapping**: For DELETE requests
- **@PatchMapping**: For PATCH requests

```java
// Instead of:
@RequestMapping(path = "/fetchUser", method = RequestMethod.GET)

// Use:
@GetMapping("/fetchUser")
```

## Parameter Binding Annotations

### @RequestParam
- **Purpose**: Binds request parameters (after ?) to method parameters
- **URL Format**: `/api/fetchUser?firstName=John&lastName=Doe&age=25`
- **Attributes**:
    - `value` or `name`: Parameter name
    - `required`: Whether parameter is mandatory (default: true)
    - `defaultValue`: Default value if not provided

**Example:**
```java
@GetMapping("/fetchUser")
public String getUser(
    @RequestParam(name = "firstName") String firstName,  // Required
    @RequestParam(name = "lastName", required = false) String lastName,  // Optional
    @RequestParam(name = "age", defaultValue = "18") int age  // With default
) {
    return "User: " + firstName + " " + lastName + ", Age: " + age;
}
```

**Type Conversion:**
- Framework automatically converts String to:
    - Primitive types (int, boolean, etc.)
    - Wrapper classes (Integer, Boolean, etc.)
    - String, Enums
    - Custom object types (with PropertyEditor)

### @PathVariable
- **Purpose**: Extracts values from URL path itself
- **URL Format**: `/api/fetchUser/12345` (12345 is path variable)
- Used for RESTful URLs

**Example:**
```java
@GetMapping("/fetchUser/{userId}")
public String getUser(@PathVariable("userId") String userId) {
    return "Fetching user with ID: " + userId;
}

// Multiple path variables
@GetMapping("/users/{userId}/posts/{postId}")
public String getPost(
    @PathVariable String userId,
    @PathVariable String postId
) {
    return "User: " + userId + ", Post: " + postId;
}
```

### @RequestBody
- **Purpose**: Binds HTTP request body (JSON/XML) to Java object
- Typically used with POST/PUT requests
- Uses Jackson/Gson libraries for JSON to Java conversion

**Example:**
```java
// User class
public class User {
    private String username;
    
    @JsonProperty("user_email")  // Maps JSON key to different field name
    private String email;
    
    // Getters and setters
}

// Controller method
@PostMapping("/saveUser")
public String saveUser(@RequestBody User user) {
    // user object automatically populated from JSON
    return "Saved: " + user.getUsername();
}

// JSON Request Body:
{
    "username": "john",
    "user_email": "john@example.com"
}
```

## Custom Type Conversion

### @InitBinder with PropertyEditor
- **Purpose**: Preprocessing of request parameters before binding
- Useful for custom formatting, validation, or transformation

**Example - Converting firstName to lowercase:**
```java
@RestController
public class SampleController {
    
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(
            String.class,
            "firstName",
            new FirstNamePropertyEditor()
        );
    }
    
    @GetMapping("/fetchUser")
    public String getUser(@RequestParam String firstName) {
        return "User: " + firstName;  // Will be lowercase
    }
}

// Custom Property Editor
public class FirstNamePropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) {
        setValue(text.trim().toLowerCase());
    }
}
```

## Response Handling

### ResponseEntity
- **Purpose**: Represents complete HTTP response (status + headers + body)
- More control than just returning object
- Can set custom status codes and headers

**Components of HTTP Response:**
- Status Code (200, 404, 500, etc.)
- Headers
- Body

**Example:**
```java
@GetMapping("/fetchUser")
public ResponseEntity<String> getUser() {
    String userData = "User details";
    
    // Success response
    return ResponseEntity
        .ok()  // Status 200
        .header("Custom-Header", "value")
        .body(userData);
    
    // Alternative syntax
    return ResponseEntity
        .status(HttpStatus.OK)
        .body(userData);
    
    // Error response
    return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body("User not found");
}
```

## How Annotations Work Together

### Request Flow Example:
```java
@RestController
@RequestMapping("/api/v1")
public class UserController {
    
    // GET: /api/v1/users?name=John&age=25
    @GetMapping("/users")
    public ResponseEntity<String> getUsers(
        @RequestParam String name,
        @RequestParam(required = false) Integer age
    ) {
        return ResponseEntity.ok("Found users");
    }
    
    // GET: /api/v1/users/123
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = findUserById(id);
        return ResponseEntity.ok(user);
    }
    
    // POST: /api/v1/users (with JSON body)
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User saved = saveUser(user);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(saved);
    }
}
```

## Key Differences Summary

| Annotation | Purpose | Key Point |
|------------|---------|-----------|
| @Controller | Mark class as controller | Returns view names by default |
| @RestController | Mark class as REST controller | Returns HTTP responses |
| @RequestMapping | Map requests to methods | Can specify method type |
| @GetMapping, etc. | Specific HTTP method mapping | Cleaner than @RequestMapping |
| @RequestParam | Bind query parameters | After ? in URL |
| @PathVariable | Bind path segments | Part of URL path |
| @RequestBody | Bind request body | For JSON/XML payloads |
| @ResponseBody | Return HTTP response | Not needed with @RestController |
| ResponseEntity | Complete response control | Status + Headers + Body |

## Best Practices

1. **Use @RestController** for REST APIs instead of @Controller + @ResponseBody
2. **Use specific mappings** (@GetMapping) instead of generic @RequestMapping
3. **Use ResponseEntity** when you need control over status codes
4. **Make optional parameters** explicitly optional with `required = false`
5. **Use @RequestBody** for complex objects in POST/PUT requests
6. **Use @PathVariable** for RESTful resource identifiers
7. **Use @RequestParam** for filtering/query parameters

## Interview Key Points

1. **@RestController = @Controller + @ResponseBody**
2. **Request parameters** come after ? in URL; **path variables** are part of the URL path
3. **DispatcherServlet** uses handler mapping to find the right controller
4. **Type conversion** happens automatically for common types
5. **ResponseEntity** gives full control over HTTP response
6. **@InitBinder** allows custom preprocessing of parameters
7. Without **@ResponseBody**, controller methods return view names (for MVC)
8. **Jackson/Gson** libraries handle JSON to Java conversion