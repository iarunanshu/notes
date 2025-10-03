# Spring Boot Filters vs Interceptors

## One-Line Definitions

**Filter**: Intercepts HTTP request and response **before** they reach the servlet

**Interceptor**: Spring-specific, intercepts HTTP request and response **after** servlet but **before** controller

## Request Processing Pipeline

### Complete Flow Diagram

```
Client Request
    ↓
Servlet Container (Tomcat)
    ↓
[FILTERS] ← Generic, servlet-agnostic
    ↓
Servlet (DispatcherServlet in Spring)
    ↓
[INTERCEPTORS] ← Spring-specific
    ↓
Controller
    ↓
Business Logic
    ↓
Response (reverse order)
```

### Detailed Architecture

```
HTTP Request
    ↓
Servlet Container
    ├── Filter 1
    ├── Filter 2
    └── Filter N
        ↓
    Servlet Selection
    ├── REST API Servlet (DispatcherServlet)
    │   ├── Interceptor 1
    │   ├── Interceptor 2
    │   └── Controller
    ├── SOAP API Servlet
    └── File Upload Servlet
```

## What is a Servlet?

**Definition**: A Java class that accepts incoming requests, processes them, and returns responses

### Multiple Servlets Example (Monolithic)
- REST API Servlet → handles `/api/*`
- SOAP Servlet → handles `/soap/*`
- File Upload Servlet → handles `/upload/*`
- Static Content Servlet → handles `/static/*`

### Modern Spring Boot (Microservices)
- Usually just **DispatcherServlet** → handles `/**` (everything)
- Configurable to specific paths like `/api/*`

## Filters

### Characteristics
- **Servlet container level** (not Spring-specific)
- Works with **any servlet framework**
- Access to raw **HttpServletRequest/Response**
- **Generic logic** applicable to all servlets
- Part of **Java EE specification**

### Implementation

```java
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@Component
public class MyFilter1 implements Filter {
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // Called once when filter is created
        System.out.println("Filter1 initialized");
    }
    
    @Override
    public void doFilter(ServletRequest request, 
                        ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        System.out.println("MyFilter1: Before processing");
        
        // Process the rest of the chain
        chain.doFilter(request, response);
        
        System.out.println("MyFilter1: After processing");
    }
    
    @Override
    public void destroy() {
        // Cleanup when filter is destroyed
        System.out.println("Filter1 destroyed");
    }
}
```

### Registering Multiple Filters with Order

```java
@Configuration
public class FilterConfig {
    
    @Bean
    public FilterRegistrationBean<MyFilter1> filter1Registration() {
        FilterRegistrationBean<MyFilter1> registration = new FilterRegistrationBean<>();
        registration.setFilter(new MyFilter1());
        registration.addUrlPatterns("/*");
        registration.setOrder(1);  // Lower number = higher priority
        return registration;
    }
    
    @Bean
    public FilterRegistrationBean<MyFilter2> filter2Registration() {
        FilterRegistrationBean<MyFilter2> registration = new FilterRegistrationBean<>();
        registration.setFilter(new MyFilter2());
        registration.addUrlPatterns("/*");
        registration.setOrder(2);
        return registration;
    }
}
```

## Interceptors

### Characteristics
- **Spring MVC level** (Spring-specific)
- Works only with **Spring framework**
- Access to **Spring context and beans**
- **Application-specific logic**
- Access to **handler methods and ModelAndView**

### Implementation (Recap)

```java
@Component
public class MyInterceptor1 implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) {
        System.out.println("Interceptor1: PreHandle");
        return true;  // false stops execution
    }
    
    @Override
    public void postHandle(HttpServletRequest request, 
                         HttpServletResponse response, 
                         Object handler, 
                         ModelAndView modelAndView) {
        System.out.println("Interceptor1: PostHandle");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler, 
                              Exception ex) {
        System.out.println("Interceptor1: AfterCompletion");
    }
}
```

### Registering Multiple Interceptors

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    
    @Autowired
    private MyInterceptor1 interceptor1;
    
    @Autowired
    private MyInterceptor2 interceptor2;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // Order is determined by registration sequence
        registry.addInterceptor(interceptor1)
                .addPathPatterns("/api/**");
                
        registry.addInterceptor(interceptor2)
                .addPathPatterns("/api/**");
    }
}
```

## Execution Order with Both Filters and Interceptors

### Request Flow (Incoming)
```
1. Filter1 - doFilter (before chain)
2. Filter2 - doFilter (before chain)
3. Interceptor1 - preHandle
4. Interceptor2 - preHandle
5. Controller Method
```

### Response Flow (Outgoing)
```
6. Interceptor2 - postHandle
7. Interceptor1 - postHandle
8. Interceptor2 - afterCompletion
9. Interceptor1 - afterCompletion
10. Filter2 - doFilter (after chain)
11. Filter1 - doFilter (after chain)
```

### Complete Example Output

```
MyFilter1: Inside
MyFilter2: Inside
Inside PreHandle of Interceptor1
Inside PreHandle of Interceptor2
Hitting Controller: Doing something
Inside PostHandle of Interceptor2
Inside PostHandle of Interceptor1
Inside AfterCompletion of Interceptor2
Inside AfterCompletion of Interceptor1
MyFilter2: Completed
MyFilter1: Completed
```

## When to Use What?

### Use Filters When:

1. **Generic security** (Spring Security uses filters)
   ```java
   // Authentication that applies to all servlets
   public class AuthenticationFilter implements Filter {
       // Check tokens, headers, etc.
   }
   ```

2. **Request/Response modification** at HTTP level
   ```java
   // CORS, compression, encoding
   public class CorsFilter implements Filter {
       // Add CORS headers
   }
   ```

3. **Logging all HTTP traffic**
   ```java
   public class RequestLoggingFilter implements Filter {
       // Log all requests regardless of framework
   }
   ```

4. **Working with non-Spring servlets**

### Use Interceptors When:

1. **Application-specific logic**
   ```java
   // Business logic validation
   public class BusinessValidationInterceptor implements HandlerInterceptor {
       // Validate based on Spring beans
   }
   ```

2. **Controller-specific processing**
   ```java
   // API versioning, rate limiting per endpoint
   public class ApiVersionInterceptor implements HandlerInterceptor {
       // Check API version for specific controllers
   }
   ```

3. **Need Spring context access**
   ```java
   @Autowired
   private UserService userService;  // Can inject Spring beans
   ```

4. **Working with ModelAndView**
   ```java
   public void postHandle(..., ModelAndView modelAndView) {
       // Modify view or model
   }
   ```

## Key Differences Table

| Aspect | Filter | Interceptor |
|--------|--------|-------------|
| **Level** | Servlet Container | Spring MVC |
| **Specification** | Java EE/Jakarta EE | Spring Framework |
| **Configuration** | `@WebFilter` or `FilterRegistrationBean` | `WebMvcConfigurer` |
| **Applies to** | All servlets | Specific servlet (DispatcherServlet) |
| **Spring Context** | No direct access | Full access |
| **Can stop chain** | Yes (`chain.doFilter()`) | Yes (`return false`) |
| **Access to** | Request/Response only | Handler, ModelAndView, Spring beans |
| **Use case** | Generic, cross-cutting | Application-specific |

## Important Notes

### PreHandle Return Value
```java
public boolean preHandle(...) {
    if (someCondition) {
        return false;  // Stops execution - no controller, no next interceptor
    }
    return true;  // Continue to next interceptor/controller
}
```

### Filter Chain
```java
public void doFilter(..., FilterChain chain) {
    // Pre-processing
    
    chain.doFilter(request, response);  // MUST call to continue
    
    // Post-processing
}
```

## Common Interview Questions

### Q1: Can you change headers in both?
**Answer**: Yes, both have access to HttpServletRequest/Response. But:
- Filters: Better for generic header manipulation (CORS, security)
- Interceptors: Better for application-specific headers

### Q2: Which executes first?
**Answer**: Filters always execute before Interceptors
```
Request → Filters → Servlet → Interceptors → Controller
```

### Q3: Can you use both together?
**Answer**: Yes, commonly used together:
- Filters for security, CORS, compression
- Interceptors for logging, validation, rate limiting

### Q4: What if preHandle returns false?
**Answer**:
- Controller not invoked
- Next interceptors not invoked
- postHandle not invoked
- afterCompletion IS invoked (for cleanup)

### Q5: Spring Security uses what?
**Answer**: Filters - because security should apply before servlet selection

## Best Practices

1. **Don't duplicate logic** - Choose appropriate level
2. **Filters for infrastructure** - Security, CORS, encoding
3. **Interceptors for business** - Validation, logging, metrics
4. **Order matters** - Plan execution sequence carefully
5. **Handle exceptions properly** - Especially in filters
6. **Keep them lightweight** - Avoid heavy processing
7. **Use appropriate scope** - Don't use interceptors for non-Spring logic

## Key Takeaways

1. **Filters = Servlet level**, Interceptors = Spring MVC level
2. **Filters are generic**, Interceptors are Spring-specific
3. **Both can modify request/response** but at different stages
4. **Execution order**: Filter1 → Filter2 → Interceptor1 → Interceptor2 → Controller → (reverse)
5. **Choose based on scope**: Generic (Filter) vs Application-specific (Interceptor)
6. **Spring Security uses Filters** because it needs to work before servlet selection
7. **Both are useful** - Often used together in real applications