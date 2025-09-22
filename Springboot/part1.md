# Spring Boot Introduction - Complete Notes

## Overview
- Spring Boot provides a quick way to create production-ready applications
- Built on top of Spring Framework (Spring MVC)
- Supports **convention over configuration**
- Uses default values for configuration (can be overridden if needed)

## Evolution Timeline
**Servlets (2013-2016) → Spring MVC → Spring Boot**

## 1. Servlets and Servlet Containers

### What is a Servlet?
- Java class that handles client requests, processes them, and returns responses
- Foundation for building web applications
- Spring and Spring Boot internally use the same concept

### Servlet Structure
```java
@WebServlet("/demoServlet1")
public class DemoServlet1 extends HttpServlet {
    // Can have only ONE of each:
    - doGet()
    - doPost()
    - doPut()
    - doDelete()
}
```

### Servlet Container
- Manages servlets (e.g., Tomcat)
- Uses **web.xml** for servlet mapping
- Application deployed as WAR file

### Problems with Servlets
1. **Complex web.xml configuration**
    - Becomes too large with hundreds of servlets
    - Contains mappings, filters, and other configurations
    - Difficult to manage and understand

2. **Tight coupling**
    - Hard to write unit tests
    - Cannot easily mock dependencies

3. **Difficult REST API management**
    - Each servlet limited to one GET, POST, PUT, DELETE method
    - Path mapping through if-else chains makes code complex

## 2. Spring Framework / Spring MVC

### Key Improvements over Servlets

#### 1. **Annotation-based Configuration**
- Removed web.xml completely
- Uses annotations like `@Controller`, `@RequestMapping`, `@GetMapping`
- More organized and readable code

#### 2. **Inversion of Control (IoC) / Dependency Injection**
- **Definition**: Flexible way to manage object dependencies and lifecycle
- Spring manages object creation and dependencies
- Uses `@Component` and `@Autowired` annotations

**Benefits of IoC:**
- Loose coupling between classes
- Easy unit testing with mock objects
- Spring resolves dependencies automatically

#### 3. **Better REST API Handling**
```java
@Controller
public class PaymentController {
    @GetMapping("/payment/endpoint1")
    public String method1() { }
    
    @GetMapping("/payment/endpoint2")
    public String method2() { }
}
```
- Multiple endpoints in single class
- Clean, organized structure
- No if-else chains needed

#### 4. **DispatcherServlet**
- Acts as **Front Controller**
- All requests go through DispatcherServlet first
- Uses **Handler Mapping** to determine:
    - Which controller to invoke
    - Which method to call

### Spring MVC Request Flow
1. Request → Servlet Container (Tomcat)
2. → DispatcherServlet
3. → Handler Mapping (finds controller)
4. → Creates controller instance
5. → IoC resolves dependencies
6. → Invokes controller method
7. → Returns response

### Spring MVC Requirements
- **pom.xml**: Define dependencies with versions
- **AppConfig class**: Configuration with `@EnableWebMvc`, `@ComponentScan`
- **DispatcherServlet class**: Extend and configure
- **Controller classes**: Business logic

## 3. Spring Boot Advantages

### Why Spring Boot?
Spring Boot solves three major challenges with Spring MVC:

### 1. **Dependency Management**

**Spring MVC Problem:**
- Must specify each dependency separately
- Must provide versions for each
- Version compatibility issues between dependencies

**Spring Boot Solution:**
- Uses **starter dependencies** (e.g., `spring-boot-starter-web`, `spring-boot-starter-test`)
- No version specification needed (except parent)
- Automatically loads compatible versions
- Internally manages all required dependencies

```xml
<!-- Spring Boot pom.xml -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- No version needed! -->
    </dependency>
</dependencies>
```

### 2. **Auto Configuration**

**Spring MVC Problem:**
- Manual configuration of DispatcherServlet
- Manual component scanning setup
- Multiple configuration classes needed

**Spring Boot Solution:**
- Single `@SpringBootApplication` annotation includes:
    - `@EnableWebMvc`
    - `@ComponentScan` (default from main class package)
    - DispatcherServlet configuration
- **Opinionated defaults** - Spring Boot provides sensible defaults
- Can override defaults if needed

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 3. **Embedded Server**

**Spring MVC Problem:**
- Create WAR file
- Deploy to external Tomcat/servlet container
- Manage server separately

**Spring Boot Solution:**
- **Embedded Tomcat** (or Jetty/Undertow)
- No WAR creation needed
- No deployment needed
- Just run the application
- Server starts automatically

## Spring Boot Characteristics

### Convention over Configuration
- Uses default values for most configurations
- Developer can override when needed
- Reduces boilerplate code significantly

### Opinionated Nature
- Spring Boot has "opinions" (default choices)
- Example: Component scanning starts from main class package
- Accept defaults or customize as needed

### Production Ready
- Quick application setup
- Built-in features for production
- Minimal configuration required

## Simple Spring Boot Application Structure

```java
// Controller
@Controller
public class MyController {
    @GetMapping("/myApi/firstApi")
    public String hello() {
        return "Hello from Concept & Coding";
    }
}

// Main Application
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**pom.xml:**
- Parent starter
- Minimal dependencies
- No version management needed

## Key Takeaways

1. **Spring Boot = Spring MVC + Auto Configuration + Embedded Server + Dependency Management**

2. **Evolution Benefits:**
    - Servlets → Spring MVC: Removed web.xml, added IoC, better API handling
    - Spring MVC → Spring Boot: Simplified configuration, embedded server, easier dependency management

3. **Development Speed:**
    - Minimal configuration
    - Quick startup
    - Focus on business logic rather than configuration

4. **Best For:**
    - Microservices
    - RESTful APIs
    - Quick prototypes
    - Production-ready applications

## Interview Key Points
- Spring Boot is built on top of Spring Framework
- Main advantages: Embedded server, Auto-configuration, Dependency management
- Uses convention over configuration principle
- DispatcherServlet still exists but is auto-configured
- IoC/Dependency Injection still fundamental
- Can override any default configuration when needed