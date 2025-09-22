# Spring Boot @Profile Annotation - Complete Notes

## Prerequisites
- Understanding of @ConditionalOnProperty annotation
- Knowledge of Spring Boot configuration management
- Familiarity with application.properties

## The Core Problem: Environment-Specific Configuration

### Real-World Scenario
Different environments require different configurations:

```
Development Environment:
- Database: username=dev, password=dev123
- Port: 45613
- Timeout: 1000ms
- URL: localhost

QA Environment:
- Database: username=qa, password=qa456
- Port: 45613
- Timeout: 500ms
- URL: qa-server.com

Production Environment:
- Database: username=prod, password=secure#789
- Port: 65121
- Timeout: 50ms
- URL: prod-server.com
```

### Configuration Differences Across Environments
1. **Database Credentials** (username/password)
2. **URLs and Port Numbers**
3. **Connection/Request Timeouts**
4. **Throttle Values**
5. **Retry Mechanisms**
6. **Resource Allocations**

## What is Profiling in Spring Boot?

**Definition**: Profiling is a mechanism to segregate configuration and beans based on different environments (profiles)

**Profile = Environment** in Spring Boot context

### Profile-Specific Property Files

```
application.properties           # Default/Common configuration
application-dev.properties       # Development specific
application-qa.properties        # QA specific
application-prod.properties      # Production specific
application-{profileName}.properties  # Custom profile
```

## Setting Up Profiles

### 1. Create Profile-Specific Property Files

**application.properties** (Default):
```properties
username=default-user
password=default-pass
spring.profiles.active=dev  # Set default profile
```

**application-dev.properties**:
```properties
username=dev-user
password=dev-pass
connection.timeout=1000
server.port=8080
```

**application-qa.properties**:
```properties
username=qa-user
password=qa-pass
connection.timeout=500
server.port=8081
```

**application-prod.properties**:
```properties
username=prod-user
password=prod-pass
connection.timeout=50
server.port=443
```

### 2. Parent-Child Configuration Hierarchy

```
application.properties (Parent)
    ├── Common configurations
    └── Default values

application-{profile}.properties (Child)
    ├── Profile-specific overrides
    └── Additional configurations
```

**Resolution Rules**:
1. If property exists in both → **Child wins**
2. If only in parent → Parent value used
3. If only in child → Child value used
4. If in neither → Application fails (unless default provided)

## How to Activate Profiles

### Method 1: Static Configuration
```properties
# In application.properties
spring.profiles.active=qa
```

### Method 2: Command Line (Runtime)
```bash
# Direct command
mvn spring-boot:run -Dspring-boot.run.profiles=prod

# Or with java -jar
java -jar myapp.jar --spring.profiles.active=prod
```

### Method 3: Maven POM Configuration
```xml
<profiles>
    <profile>
        <id>local</id>
        <properties>
            <spring-boot.run.profiles>dev</spring-boot.run.profiles>
        </properties>
    </profile>
    <profile>
        <id>staging</id>
        <properties>
            <spring-boot.run.profiles>qa</spring-boot.run.profiles>
        </properties>
    </profile>
    <profile>
        <id>production</id>
        <properties>
            <spring-boot.run.profiles>prod</spring-boot.run.profiles>
        </properties>
    </profile>
</profiles>
```

**Usage**:
```bash
mvn spring-boot:run -P production
```

### Method 4: Environment Variables
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar myapp.jar
```

## @Profile Annotation

### Definition
Creates beans conditionally based on active profile(s)

### Basic Usage
```java
@Component
@Profile("prod")  // Bean created only when 'prod' profile is active
public class MySQLConnection {
    public MySQLConnection() {
        System.out.println("MySQL connection initialized for production");
    }
}

@Component
@Profile("dev")  // Bean created only when 'dev' profile is active
public class NoSQLConnection {
    public NoSQLConnection() {
        System.out.println("NoSQL connection initialized for development");
    }
}
```

### Example with Configuration Values
```java
@Component
public class DatabaseConfig {
    
    @Value("${username}")
    private String username;
    
    @Value("${password}")
    private String password;
    
    @PostConstruct
    public void init() {
        System.out.println("Username: " + username);
        System.out.println("Password: " + password);
    }
}
```

### Profile Execution Flow

**Scenario 1: Profile = QA**
```properties
spring.profiles.active=qa
```

```java
@Component
@Profile("prod")
public class MySQLConnection { }  // NOT created (prod ≠ qa)

@Component
@Profile("dev")
public class NoSQLConnection { }  // NOT created (dev ≠ qa)

@Component
@Profile("qa")
public class QAConnection { }     // CREATED (qa = qa)
```

## Multiple Profiles

### Setting Multiple Profiles
```properties
# Both profiles are active
spring.profiles.active=prod,qa

# For properties: Last one wins (qa properties used)
# For beans: Both profile beans created
```

### Using Multiple Profiles in Annotation
```java
@Component
@Profile({"dev", "qa"})  // Created for EITHER dev OR qa
public class NonProdDatabase { }

@Component
@Profile("!prod")  // Created for any profile EXCEPT prod
public class DebugLogger { }
```

## @Profile vs @ConditionalOnProperty

### The Interview Question
**Q**: "Two applications sharing common codebase - how to ensure bean created only for one application?"

### Answer Comparison

#### Using @ConditionalOnProperty (✅ CORRECT)
```java
// Common codebase
@Component
@ConditionalOnProperty(
    name = "feature.app1.enabled",
    havingValue = "true"
)
public class App1SpecificBean { }
```

```properties
# App1's application.properties
feature.app1.enabled=true

# App2's application.properties
feature.app1.enabled=false
```

#### Using @Profile (❌ TECHNICALLY INCORRECT)
```java
// Common codebase
@Component
@Profile("app1")
public class App1SpecificBean { }
```

```properties
# App1's application.properties
spring.profiles.active=app1  # Not an environment!

# App2's application.properties
spring.profiles.active=app2  # Not an environment!
```

### Why @Profile is Wrong for This Use Case

1. **Profiles represent ENVIRONMENTS**, not applications
2. **Semantic violation**: app1, app2 are not environments
3. **Confusion**: Creates files like `application-app1.properties` (meaningless)
4. **Maintenance issues**: Mixing application logic with environment segregation

### Correct Usage Patterns

**@Profile for**:
- Environment segregation (dev, qa, prod)
- Environment-specific beans
- Environment-specific configurations

**@ConditionalOnProperty for**:
- Feature toggles
- Application-specific beans
- Conditional bean creation based on business logic

## Best Practices

### 1. Profile Naming Convention
```properties
# Good - Clear environment names
spring.profiles.active=development
spring.profiles.active=staging
spring.profiles.active=production

# Bad - Unclear purpose
spring.profiles.active=config1
spring.profiles.active=app-feature
```

### 2. Property Organization
```properties
# application.properties - Common/default values
server.servlet.context-path=/api
logging.level.root=INFO

# application-dev.properties - Dev overrides
logging.level.root=DEBUG
spring.jpa.show-sql=true

# application-prod.properties - Prod overrides
logging.level.root=ERROR
spring.jpa.show-sql=false
```

### 3. Profile-Specific Beans
```java
@Configuration
@Profile("dev")
public class DevSecurityConfig {
    // Relaxed security for development
}

@Configuration
@Profile("prod")
public class ProdSecurityConfig {
    // Strict security for production
}
```

### 4. Testing with Profiles
```java
@SpringBootTest
@ActiveProfiles("test")
class MyServiceTest {
    // Uses application-test.properties
}
```

## Common Interview Questions

**Q1: What happens if no profile is set?**
- Uses `application.properties` only
- No profile-specific properties loaded
- Beans without @Profile annotation created

**Q2: How does property resolution work with profiles?**
- Profile-specific properties override default
- Last profile wins for property files
- All matching profile beans are created

**Q3: Can you use profiles for feature flags?**
- Technically possible but **not recommended**
- Use @ConditionalOnProperty instead
- Profiles are for environments, not features

**Q4: What's the execution order priority?**
```
1. Command line arguments
2. Environment variables
3. application-{profile}.properties
4. application.properties
```

## Priority Order Summary

When multiple methods set profiles:
1. **Command Line** (Highest priority)
2. **Environment Variables**
3. **application.properties**
4. **Default** (if nothing set)

## Key Takeaways

1. **Profiles = Environments** (dev, qa, prod)
2. **@Profile** creates beans conditionally based on active profile
3. **Don't use @Profile** for application-specific logic
4. **Multiple profiles** can be active simultaneously
5. **Profile-specific properties** override default properties
6. **Runtime profile selection** overrides static configuration

## Common Pitfalls

1. **Using profiles for non-environment purposes**
2. **Forgetting default values** in application.properties
3. **Not understanding property resolution hierarchy**
4. **Mixing @Profile with @ConditionalOnProperty** inappropriately