# Spring Boot HATEOAS (Hypermedia as the Engine of Application State)

## What is HATEOAS?

**Definition**: HATEOAS is a REST architectural constraint where the server provides information about available next actions/APIs in the response itself, telling the client what operations can be performed after the current step.

**Key Concept**: The client doesn't need to hardcode the API endpoints or business logic about which API to call next - the server provides this information dynamically.

## Example Use Case

### User Registration Flow

```
POST /addUser (User created - unverified)
    ├── GET /user/{id} (Get user details)
    ├── DELETE /user/{id} (Delete user)
    └── Verify User
        ├── SMS Verify
        │   ├── POST /sms-verify-start
        │   └── POST /sms-verify-finish
        └── Email Verify
            ├── POST /email-verify-start
            └── POST /email-verify-finish
```

## Response Structure Comparison

### Without HATEOAS

```json
{
    "userId": "123456",
    "name": "John Doe",
    "verifyStatus": "unverified"
}
```

### With HATEOAS

```json
{
    "userId": "123456",
    "name": "John Doe",
    "verifyStatus": "unverified",
    "links": [
        {
            "rel": "self",
            "href": "/api/user/123456",
            "type": "GET"
        },
        {
            "rel": "verify",
            "href": "/api/sms-verify-start/123456",
            "type": "POST"
        }
    ]
}
```

## Two Major Purposes of HATEOAS

### 1. Loose Coupling
- Reduces dependency between client and server
- Client doesn't need to know business logic about API flow
- Server controls the workflow

### 2. API Discovery
- Client doesn't need to know all available APIs upfront
- Server provides available actions based on current state
- Dynamic navigation through API endpoints

## When NOT to Use HATEOAS

### ❌ Bad Practice: Adding ALL Possible Links

```json
{
    "userId": "123456",
    "name": "John Doe",
    "verifyStatus": "unverified",
    "links": [
        {"rel": "self", "href": "/api/user/123456", "type": "GET"},
        {"rel": "delete", "href": "/api/user/123456", "type": "DELETE"},
        {"rel": "update", "href": "/api/user/123456", "type": "PATCH"},
        {"rel": "sms-verify-start", "href": "/api/sms-verify-start/123456", "type": "POST"},
        {"rel": "sms-verify-finish", "href": "/api/sms-verify-finish/123456", "type": "POST"},
        {"rel": "email-verify-start", "href": "/api/email-verify-start/123456", "type": "POST"},
        {"rel": "email-verify-finish", "href": "/api/email-verify-finish/123456", "type": "POST"}
        // ... many more links
    ]
}
```

**Problems:**
- Bloated API response
- Increased payload size
- Higher latency
- Complex server-side logic
- Performance impact

## When TO Use HATEOAS

### ✅ Good Practice: Add Only Contextually Relevant Links

### Example: Tight Coupling (Without HATEOAS)

#### Server Response
```json
{
    "userId": "123456",
    "name": "John Doe",
    "verifyStatus": "unverified",
    "verifyType": "SMS",
    "verifyState": "NOT_STARTED"
}
```

#### Client Code (Tight Coupling)
```javascript
// Client needs to know all business logic
if (response.verifyStatus === "unverified") {
    if (response.verifyType === "SMS") {
        if (response.verifyState === "NOT_STARTED") {
            callAPI("/api/sms-verify-start/" + response.userId);
        } else if (response.verifyState === "STARTED") {
            callAPI("/api/sms-verify-finish/" + response.userId);
        }
    } else if (response.verifyType === "EMAIL") {
        if (response.verifyState === "NOT_STARTED") {
            callAPI("/api/email-verify-start/" + response.userId);
        } else if (response.verifyState === "STARTED") {
            callAPI("/api/email-verify-finish/" + response.userId);
        }
    }
}
```

### Example: Loose Coupling (With HATEOAS)

#### Server Response
```json
{
    "userId": "123456",
    "name": "John Doe",
    "verifyStatus": "unverified",
    "links": [
        {
            "rel": "verify",
            "href": "/api/sms-verify-start/123456",
            "type": "POST"
        }
    ]
}
```

#### Client Code (Loose Coupling)
```javascript
// Client just uses the provided link
if (response.verifyStatus === "unverified") {
    const verifyLink = response.links.find(link => link.rel === "verify");
    callAPI(verifyLink.href, verifyLink.type);
}
```

## Implementation in Spring Boot

### 1. Add Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

### 2. Create Base Response with Links

```java
import org.springframework.hateoas.Link;
import java.util.ArrayList;
import java.util.List;

public class HateoasLinks {
    private List<Link> links = new ArrayList<>();
    
    public void addLink(Link link) {
        this.links.add(link);
    }
    
    public List<Link> getLinks() {
        return links;
    }
}
```

### 3. Create Response Model

```java
public class UserResponse extends HateoasLinks {
    private String userId;
    private String name;
    private String verifyStatus;
    
    // Constructors, getters, setters
}
```

### 4. Controller Implementation

```java
import org.springframework.hateoas.Link;
import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

@RestController
@RequestMapping("/api")
public class UserController {
    
    @PostMapping("/addUser")
    public UserResponse addUser(@RequestBody UserRequest request) {
        // Create user and get response
        UserResponse response = userService.addUser(request);
        
        // Add HATEOAS links based on business logic
        addRelevantLinks(response);
        
        return response;
    }
    
    private void addRelevantLinks(UserResponse response) {
        // Method 1: Using WebMvcLinkBuilder
        if ("unverified".equals(response.getVerifyStatus())) {
            Link verifyLink = linkTo(UserController.class)
                .slash("sms-verify-start")
                .slash(response.getUserId())
                .withRel("verify")
                .withType("POST");
            
            response.addLink(verifyLink);
        }
        
        // Method 2: Using Link.of()
        Link selfLink = Link.of("/api/user/" + response.getUserId())
            .withRel("self")
            .withType("GET");
        
        response.addLink(selfLink);
    }
}
```

### 5. Two Ways to Create Links

#### Method 1: WebMvcLinkBuilder

```java
Link link = linkTo(UserController.class)
    .slash("sms-verify-finish")
    .slash(userId)
    .withRel("verify")
    .withType("POST");
```

#### Method 2: Link.of()

```java
Link link = Link.of("/api/sms-verify-finish/" + userId)
    .withRel("verify")
    .withType("POST");
```

## Best Practices

### 1. Add Links Cautiously
```java
private void addLinks(UserResponse response) {
    // Only add contextually relevant links
    if (shouldAddVerifyLink(response)) {
        response.addLink(createVerifyLink(response));
    }
    
    // Always add self link
    response.addLink(createSelfLink(response));
}
```

### 2. Dynamic Link Generation
```java
private Link createVerifyLink(UserResponse response) {
    String verifyEndpoint = determineVerifyEndpoint(response);
    return Link.of(verifyEndpoint)
        .withRel("verify")
        .withType("POST");
}

private String determineVerifyEndpoint(UserResponse response) {
    // Business logic to determine correct endpoint
    if ("SMS".equals(response.getVerifyType())) {
        if ("NOT_STARTED".equals(response.getVerifyState())) {
            return "/api/sms-verify-start/" + response.getUserId();
        } else {
            return "/api/sms-verify-finish/" + response.getUserId();
        }
    }
    // ... more logic
}
```

### 3. Standard Relations

```java
public class StandardRelations {
    public static final String SELF = "self";        // Current resource
    public static final String NEXT = "next";        // Next in sequence
    public static final String PREV = "previous";    // Previous in sequence
    public static final String FIRST = "first";      // First in collection
    public static final String LAST = "last";        // Last in collection
    public static final String COLLECTION = "collection"; // Parent collection
}
```

## Advantages vs Disadvantages

### Advantages ✅
1. **Loose Coupling** - Client doesn't need API workflow knowledge
2. **API Discovery** - Self-documenting APIs
3. **Flexibility** - Server can change workflow without client changes
4. **Reduced Client Logic** - Business rules stay on server

### Disadvantages ❌
1. **Increased Payload Size** - Extra data in responses
2. **Server Complexity** - Dynamic link generation logic
3. **Performance Impact** - Additional processing time
4. **Learning Curve** - Developers need to understand HATEOAS

## Real-World Usage

**Important Note**: Despite criticism, HATEOAS is used by major companies:
- PayPal API
- GitHub API
- Spring Data REST
- Many enterprise applications

## Common Interview Questions

### Q1: What is HATEOAS?
**Answer**: Hypermedia as the Engine of Application State - a REST constraint where server provides links to next possible actions in API responses.

### Q2: When should you use HATEOAS?
**Answer**: When you want to:
- Achieve loose coupling between client and server
- Move business logic from client to server
- Enable API discovery
- Support dynamic workflows

### Q3: What are the disadvantages?
**Answer**:
- Increased response size
- Server-side complexity
- Performance overhead
- Not suitable for all use cases

### Q4: How does it achieve loose coupling?
**Answer**: By moving the decision logic of "what API to call next" from client to server. Client just follows the provided links without knowing the business rules.

## Key Takeaways

1. **HATEOAS tells clients what they can do next** via links in responses
2. **Don't add all possible links** - only contextually relevant ones
3. **Moves business logic to server** - achieving loose coupling
4. **Use judiciously** - analyze if loose coupling benefit outweighs complexity
5. **Standard in RESTful APIs** - Level 3 of Richardson Maturity Model
6. **Spring Boot provides excellent support** via spring-boot-starter-hateoas

## Best Practice Summary

```java
// DO THIS ✅
if (needsVerification && !alreadyVerified) {
    response.addLink(createVerifyLink());
}

// NOT THIS ❌
response.addLink(getAllPossibleLinks()); // Don't bloat response
```

Remember: **HATEOAS is a tool** - use it where it adds value, not everywhere blindly.