# Maven - Complete Notes

## Overview
- **Maven**: A project management tool (not just build management)
- Helps developers with:
    - Build generation
    - Dependency resolution
    - Documentation
    - Many other functionalities
- Uses **POM (Project Object Model)** to achieve this
- Maven looks for `pom.xml` in current directory when executing commands

## Maven vs Ant

### Ant (Before Maven)
- Had to tell **what to do** AND **how to do it**
- Required providing step-by-step sequences
- Example: For compilation, specify javac command explicitly

### Maven
- Only tell **what to do**
- Maven knows **how to do it**
- Example: Just say `mvn compile`, Maven handles javac internally
- Simplifies developer tasks significantly

## Maven Project Structure

```
app-name/
├── pom.xml
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── conceptandcoding/
│   │               └── learningspringboot/
│   │                   └── SpringBootApplication.java
│   └── test/
│       └── java/
│           └── com/
│               └── conceptandcoding/
│                   └── learningspringboot/
│                       └── SpringBootApplicationTest.java
└── target/ (created after build)
```

## POM.xml Structure

### 1. **XML Schema**
```xml
<project xmlns="..." xmlns:xsi="..." xsi:schemaLocation="...">
```
- Ensures XML adheres to correct structure
- Defines what elements can be used where

### 2. **Parent**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.0</version>
</parent>
```
- Defines parent project
- Current project inherits configurations from parent
- If no parent specified, inherits from **Super POM** (top of hierarchy)
- Every POM is ultimately a child of Super POM

### 3. **Project Coordinates**
```xml
<groupId>com.conceptandcoding</groupId>
<artifactId>learning-springboot</artifactId>
<version>0.0.1-SNAPSHOT</version>
```
- **Uniquely identifies** project in Maven Central
- groupId: Company/organization name
- artifactId: Project name
- version: Current version

### 4. **Properties**
```xml
<properties>
    <java.version>17</java.version>
</properties>
```
- Key-value pairs for configuration
- Can be referenced using `${java.version}` anywhere in POM

### 5. **Dependencies**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
- List of project dependencies
- Maven downloads from repositories

### 6. **Repositories**
```xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
</repositories>
```
- Where Maven looks for dependencies
- Default: Maven Central (even if not specified)

### 7. **Build**
```xml
<build>
    <plugins>
        <plugin>
            <!-- Plugin configuration -->
        </plugin>
    </plugins>
</build>
```
- Add custom tasks to build phases
- Configure plugins for specific phases

## Maven Build Lifecycle

### Seven Phases (Sequential)
1. **validate** → 2. **compile** → 3. **test** → 4. **package** → 5. **verify** → 6. **install** → 7. **deploy**

### Important Concepts
- **Phases**: Stages of build lifecycle
- **Goals**: Tasks within each phase (multiple goals per phase)
- **Sequential Execution**: Running a phase executes all previous phases first
- **Example**: Running `mvn package` executes validate → compile → test → package

## Build Phases in Detail

### 1. **Validate**
- **Command**: `mvn validate`
- **Purpose**: Validates project structure
- Maven doesn't enforce validation by default
- Can add custom validation plugins:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <executions>
        <execution>
            <phase>validate</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 2. **Compile**
- **Command**: `mvn compile`
- **Purpose**: Compiles source code to bytecode
- Converts `.java` files to `.class` files
- Output placed in `target/classes/`
- Internally uses `javac` (abstracted from developer)

### 3. **Test**
- **Command**: `mvn test`
- **Purpose**: Runs unit test cases
- Executes all tests in `src/test/java/`
- Runs after validate and compile phases

### 4. **Package**
- **Command**: `mvn package`
- **Purpose**: Creates JAR/WAR file
- Bundles compiled code into distributable format
- Output placed in `target/` folder
- File type depends on packaging configuration

### 5. **Verify**
- **Command**: `mvn verify`
- **Purpose**: Verify package integrity
- Can add static code analysis plugins:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <executions>
        <execution>
            <phase>verify</phase>
            <goals>
                <goal>pmd</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
- Checks for: unused variables, empty catch blocks, duplicate code

### 6. **Install**
- **Command**: `mvn install`
- **Purpose**: Installs package to **local repository**
- Local repository location:
    - Mac/Linux: `~/.m2/repository/`
    - Windows: `C:\Users\{username}\.m2\repository\`
- Follows package structure: `groupId/artifactId/version/`

### 7. **Deploy**
- **Command**: `mvn deploy`
- **Purpose**: Deploys package to **remote repository**
- Requires distribution management configuration:
```xml
<distributionManagement>
    <repository>
        <id>remote-repo</id>
        <url>https://company-repo-url.com</url>
    </repository>
</distributionManagement>
```

## Repository Types

### Local Repository
- **Location**: User's system (`~/.m2/repository/`)
- **Purpose**: Cache for dependencies
- **Configurable**: Via `settings.xml`
- Maven checks here first before remote

### Remote Repository
- **Types**:
    - Maven Central (public)
    - Company repository (private)
- **Purpose**: Central storage for artifacts
- **Access**: May require authentication

## Settings.xml Configuration

### Location
- Found in `~/.m2/settings.xml`
- Created when Maven is installed

### Key Configurations

#### 1. **Local Repository Path**
```xml
<localRepository>/custom/path/to/repository</localRepository>
```

#### 2. **Server Credentials**
```xml
<servers>
    <server>
        <id>remote-repo</id>
        <username>user</username>
        <password>pass</password>
    </server>
</servers>
```

## Dependency Resolution Flow

1. Maven checks **local repository** first
2. If not found, downloads from **remote repository**
3. Stores in local repository for future use
4. Subsequent requests use local copy

## Key Maven Commands

| Command | Purpose | Phases Executed |
|---------|---------|-----------------|
| `mvn validate` | Validate project | validate |
| `mvn compile` | Compile source code | validate → compile |
| `mvn test` | Run tests | validate → compile → test |
| `mvn package` | Create JAR/WAR | validate → compile → test → package |
| `mvn verify` | Verify package | validate → compile → test → package → verify |
| `mvn install` | Install to local repo | All up to install |
| `mvn deploy` | Deploy to remote repo | All phases |
| `mvn clean` | Clean target directory | N/A (separate lifecycle) |

## Best Practices

1. **Dependency Management**
    - Use parent POMs for version management
    - Avoid hardcoding versions in child POMs

2. **Repository Configuration**
    - Configure company repository in settings.xml
    - Use local repository to avoid repeated downloads

3. **Build Configuration**
    - Add necessary plugins in appropriate phases
    - Use properties for configuration values

4. **Project Structure**
    - Follow Maven standard directory layout
    - Keep test and main code separate

## Interview Key Points

1. **Maven is more than build tool** - It's a project management tool
2. **POM inheritance** - Every POM inherits from Super POM
3. **Build lifecycle is sequential** - Running a phase runs all previous phases
4. **Two types of repositories** - Local (system) and Remote (external)
5. **Maven vs Ant** - Maven: what to do, Ant: what and how to do
6. **Dependency resolution** - Local first, then remote
7. **Settings.xml** - Configure local repository path and credentials
8. **Distribution management** - Required for deploy phase
9. **Goals vs Phases** - Phases contain multiple goals (tasks)
10. **Build element** - Used to add custom tasks to phases