# CLAUDE.md - AI Assistant Guide

## Repository Overview

This is a **Java Memory Management educational repository** from a Udemy course. It contains 5 progressively complex modules designed to teach memory-related concepts, anti-patterns, and best practices in Java. The repository includes **intentional bugs and memory leaks** for learning purposes.

**CRITICAL:** This is a learning repository. Many bugs are intentional demonstrations of anti-patterns. Do not "fix" bugs unless explicitly requested by the user.

---

## Repository Structure

```
udemy-java-memory-management/
├── EscapingReferences/         # Module 1: Encapsulation violations
├── GarbageCollection/          # Module 2: GC demonstration
├── MemoryTest/                 # Module 3: Pass-by-value and references
├── SoftLeaks/                  # Module 4: Multi-threaded memory leaks
├── StudentManager/             # Module 5: Enterprise web app with leaks
│   ├── build.xml              # Apache Ant build configuration
│   ├── web.xml                # Servlet/web app configuration
│   ├── lib/                   # Dependencies (70+ JARs)
│   ├── web/                   # JSP pages and web resources
│   └── src/                   # Java source + Spring/Hibernate configs
├── apache-jmeter-2.12.zip     # Load testing tool (for memory profiling)
├── tomcat.zip                 # Apache Tomcat web server
└── README.md                  # Minimal project description
```

---

## Module Breakdown

### Module 1: EscapingReferences (Simple)

**Location:** `/EscapingReferences/`

**Purpose:** Demonstrates the "Escaping References" anti-pattern where internal object references are exposed, breaking encapsulation.

**Key Files:**
- `Main.java` - Entry point demonstrating the problem
- `CustomerRecords.java` - Contains the intentional bug (returns direct reference to internal HashMap)
- `Customer.java` - Simple POJO

**Intentional Bug:**
```java
public Map<String, Customer> getCustomers() {
    return this.records;  // PROBLEM: Exposes internal mutable collection
}
```

**Learning Objective:** Understanding defensive copying and encapsulation

---

### Module 2: GarbageCollection (Simple)

**Location:** `/GarbageCollection/`

**Purpose:** Demonstrates Java's garbage collection mechanism and memory monitoring.

**Key Files:**
- `Main.java` - Creates 1M objects, monitors memory before/after GC
- `Customer.java` - Simple domain object

**Key Concepts:**
- `Runtime.getRuntime().freeMemory()` for memory monitoring
- `System.gc()` for suggesting garbage collection
- Object eligibility for GC when references are lost

---

### Module 3: MemoryTest (Simple)

**Location:** `/MemoryTest/`

**Purpose:** Demonstrates Java's pass-by-value behavior with object references.

**Key Files:**
- `Main.java` - Demonstrates reference passing behavior
- `Container.java` - Simple wrapper object

**Key Concepts:**
- String immutability
- Pass-by-value (references are passed by value)
- Difference between modifying objects vs reassigning references

---

### Module 4: SoftLeaks (Intermediate)

**Location:** `/SoftLeaks/`

**Purpose:** Demonstrates a memory leak in a multi-threaded application.

**Key Files:**
- `CustomerHarness.java` - Main class, spawns 10 threads, monitors memory
- `CustomerManager.java` - **Contains intentional memory leak**
- `GenerateCustomerTask.java` - Runnable that creates customers continuously
- `Customer.java` - Domain object

**Intentional Bug:**
```java
public Customer getNextCustomer() {
    // BUG: Should remove but only retrieves
    // customers.remove(0);  // This line is commented out!
    return customers.get(0);
}
```

**Learning Objective:** How memory leaks occur when objects aren't properly released, especially in multi-threaded scenarios.

---

### Module 5: StudentManager (Advanced - Enterprise Web Application)

**Location:** `/StudentManager/`

**Purpose:** A complete Spring MVC + JPA/Hibernate web application demonstrating memory leaks in enterprise contexts.

**Architecture:** Classic 3-tier Spring MVC
```
Controllers → Services → DAO → Domain Model
```

**Key Files by Layer:**

**Domain Layer** (`com.virtualpairprogrammers.domain`):
- `Tutor.java` - JPA entity with @OneToMany and @ManyToMany relationships
- `Student.java` - JPA entity with @ManyToOne to Tutor
- `Subject.java` - JPA entity for courses
- `Address.java` - @Embeddable component
- `queries.hbm.xml` - Hibernate named queries

**DAO Layer** (`com.virtualpairprogrammers.dao`):
- `TutorDao.java` - Data access (stub implementation, DB removed for demo)

**Service Layer** (`com.virtualpairprogrammers.services`):
- `TutorManagement.java` - Business logic with @Transactional
- `UUIDGenerator.java` - **INTENTIONAL MEMORY LEAK** (static list that grows forever)
- `NoResultsFoundException.java` - Custom exception

**Controller Layer** (`com.virtualpairprogrammers.controllers`):
- `ManageTutorsController.java` - Display operations
- `AddNewTutorController.java` - Tutor creation (triggers UUID leak)

**Primary Intentional Bug:**
```java
public class UUIDGenerator {
    private static List<UUID> usedUUIDs = new ArrayList<UUID>();  // MEMORY LEAK!

    public static String newUUID() {
        while (true) {
            UUID candidate = UUID.randomUUID();
            if (!usedUUIDs.contains(candidate)) {
                usedUUIDs.add(candidate);  // Never cleared - leak grows forever!
                return candidate.toString();
            }
        }
    }
}
```

**Web Endpoints:**
- `/displayAllTutors.html` - List all tutors
- `/addNewTutor.html` - Add new tutor (triggers UUID generation)
- `/displayTutorDetail.html?id=X` - View tutor details

---

## Technologies and Frameworks

### Core Technologies

**All Modules:**
- Java SE (version not specified, but pre-Java 8 based on syntax)
- IntelliJ IDEA project structure

**StudentManager Specific:**
- **Spring Framework 3.0.3** (Core, MVC, Web, Transactions)
- **Hibernate 4.1.4.Final** (ORM) + JPA 2.0 API
- **Apache Derby** database (client driver included)
- **Apache Tomcat** (included as tomcat.zip)
- **Apache Ant** for builds
- **Servlet API 2.5** + JSP + JSTL
- **Apache JMeter 2.12** for load testing
- **Commons Libraries** (DBCP, Collections, Lang, IO, Logging)
- **SLF4J** + **Log4j** for logging
- **Jackson** for JSON
- **EhCache** for second-level caching
- **JUnit 4.x** for testing

---

## Build and Run Instructions

### Simple Modules (1-4)

**Using IntelliJ IDEA:**
1. Open project root in IntelliJ
2. Each module auto-detected via `.iml` files
3. Run the `Main` class in each module

**Using Command Line:**
```bash
# Navigate to module
cd /home/user/udemy-java-memory-management/EscapingReferences

# Compile
javac -d build src/com/udemy/memorymanagement/escapingreferences/*.java

# Run
java -cp build com.udemy.memorymanagement.escapingreferences.Main
```

### StudentManager Web Application

**Build with Ant:**
```bash
cd /home/user/udemy-java-memory-management/StudentManager
ant BuildAll
```

**Available Ant Targets:**
- `clean` - Remove build artifacts
- `prep` - Create build directories
- `compile` - Compile Java sources
- `war` - Create WAR file (`mywebapp.war`)
- `deploy` - Copy WAR to `../tomcat/webapps`
- `BuildAll` - Complete build and deploy

**Run Application:**
```bash
# Extract Tomcat first
cd /home/user/udemy-java-memory-management
unzip -q tomcat.zip

# Start Tomcat
cd tomcat/bin
java -Dsun.lang.ClassLoader.allowArraySyntax=true -XX:MaxPermSize=256M -jar bootstrap.jar

# Access at: http://localhost:8080/mywebapp/
```

**Load Testing with JMeter:**
```bash
# Extract JMeter
unzip -q apache-jmeter-2.12.zip
cd apache-jmeter-2.12/bin
./jmeter

# Create test plan to repeatedly call /addNewTutor.html
# Monitor memory growth from UUID leak
```

---

## Development Conventions

### Package Structure
- Simple modules: `com.udemy.memorymanagement.<modulename>`
- StudentManager: `com.virtualpairprogrammers.<layer>`

### Naming Conventions
- Classes: PascalCase (`CustomerManager`, `TutorManagement`)
- Methods: camelCase (`getNextCustomer`, `newUUID`)
- Variables: camelCase (`usedUUIDs`, `customers`)

### Spring Annotations Used
- `@Controller` - MVC controllers
- `@Service` - Service layer
- `@Component` - Generic components
- `@Autowired` - Dependency injection
- `@Transactional` - Transaction boundaries
- `@Entity`, `@Id`, `@OneToMany`, `@ManyToOne` - JPA entities

### Configuration Files
- `application.xml` - Spring application context (services, DAOs)
- `Dispatcher-servlet.xml` - Spring MVC configuration (controllers)
- `persistence.xml` - JPA configuration
- `queries.hbm.xml` - Hibernate named queries
- `log4j.properties` - Logging configuration
- `web.xml` - Servlet configuration

---

## Key Memory Management Concepts

### Anti-Patterns Demonstrated

1. **Escaping References**
   - Returning direct references to internal mutable collections
   - Solution: Return `Collections.unmodifiableMap()` or defensive copies

2. **Static Collection Leaks**
   - Static collections that grow indefinitely
   - Example: `UUIDGenerator.usedUUIDs`
   - Solution: Use weak references, bounded caches, or clear periodically

3. **Unbounded List Growth**
   - Collections that accumulate without removal
   - Example: `SoftLeaks.CustomerManager`
   - Solution: Implement proper cleanup logic

4. **JPA/Hibernate Memory Issues**
   - Large result sets loaded into memory
   - Circular references between entities
   - Detached entities holding onto session data
   - Solution: Use pagination, projections, proper fetching strategies

### Best Practices Taught

1. **Defensive Copying:** Return copies of mutable objects
2. **Proper Collection Management:** Remove objects when done
3. **Memory Monitoring:** Use `Runtime.getRuntime().freeMemory()`
4. **Resource Cleanup:** Especially in multi-threaded environments
5. **Fetch Strategies:** Lazy vs Eager loading in JPA
6. **Batch Sizing:** Use `@BatchSize` to optimize queries

---

## Working with This Repository

### When User Asks to "Fix Bugs"

**ASK FIRST:** Are they asking to:
1. Fix intentional teaching bugs (UUIDGenerator, CustomerManager)?
2. Fix actual implementation issues?
3. Understand why the bugs exist?

Many bugs are **intentional for learning purposes**. Confirm before making changes.

### When Adding New Features

**Respect the Architecture:**
- **StudentManager:** Follow Controller → Service → DAO pattern
- Use Spring dependency injection
- Add JPA annotations for new entities
- Update Spring XML configs for new beans (if not using component-scan)

### When Refactoring

**Preserve Learning Value:**
- Keep intentional bugs unless explicitly asked to fix
- Maintain clear separation of concerns
- Document why changes were made
- Don't introduce modern Java features (lambdas, streams) unless requested - preserve the course's Java version

### When Testing

**Memory Profiling:**
- Use `Runtime.getRuntime().freeMemory()` for basic monitoring
- Use JMeter to generate load
- Monitor heap growth over time
- Use JVM flags: `-XX:+PrintGCDetails`, `-XX:MaxPermSize=256M`

---

## Common Tasks Guide

### Adding a New Simple Module

1. Create directory: `NewModule/`
2. Create source structure: `src/com/udemy/memorymanagement/newmodule/`
3. Add `Main.java` entry point
4. Create IntelliJ module (`.iml` file) or use command-line compilation

### Adding a New Entity to StudentManager

1. Create entity class in `com.virtualpairprogrammers.domain`
2. Add JPA annotations (`@Entity`, `@Id`, etc.)
3. Define relationships to existing entities
4. Update `persistence.xml` if needed
5. Create DAO if complex queries needed
6. Add service methods in appropriate service class
7. Create controller endpoints if web access needed
8. Create JSP views for display

### Modifying Spring Configuration

**For Services/DAOs:**
- Edit `src/application.xml`
- Add `<bean>` definitions or rely on component-scan

**For Controllers:**
- Edit `src/Dispatcher-servlet.xml`
- Add view resolvers, handler mappings

**For Database:**
- Edit `persistence.xml` for JPA
- Edit `application.xml` for DataSource (currently commented out)

### Running Load Tests

1. Build and deploy StudentManager
2. Start Tomcat
3. Launch JMeter
4. Create HTTP Request sampler targeting `/addNewTutor.html`
5. Add Thread Group (e.g., 50 threads, loop forever)
6. Monitor heap with `jconsole` or `jvisualvm`
7. Observe memory growth from UUID leak

---

## Git Workflow

### Current Branch
- Working on: `claude/claude-md-mhy279ms496ox2fx-015E6nAXn28RVSUdGpAe2xYz`
- Push commands: `git push -u origin <branch-name>`

### Commit Conventions
- Keep commits focused and atomic
- Use descriptive messages
- Reference learning objectives when relevant
- Example: "Fix UUIDGenerator memory leak in StudentManager"

---

## Important Notes for AI Assistants

### DO:
- Ask clarifying questions before "fixing" intentional bugs
- Explain memory concepts when making changes
- Preserve the educational value of the code
- Follow existing architectural patterns
- Document why memory leaks occur

### DON'T:
- Automatically fix intentional teaching bugs
- Introduce modern Java features without confirmation
- Remove valuable learning examples
- Overcomplicate simple modules
- Modify Spring/Hibernate configs without understanding impact

### When Uncertain:
- Check if code has comments like "VERY BAD METHOD IMPLEMENTATION!"
- Ask: "Is this an intentional teaching bug or a real issue?"
- Explain trade-offs of proposed changes
- Provide learning context with technical changes

---

## Learning Objectives Summary

This repository teaches:

1. **Java Memory Model:** Stack vs Heap, object lifecycle
2. **Garbage Collection:** How it works, when it runs, how to monitor
3. **Memory Leaks:** Common patterns and how to identify them
4. **Reference Handling:** Pass-by-value, object references, escaping references
5. **Enterprise Memory Issues:** Static leaks, entity relationships, session management
6. **Profiling and Monitoring:** Using Runtime API, JMeter, JVM tools
7. **Best Practices:** Defensive copying, proper cleanup, fetch strategies

---

## Quick Reference

### Memory Monitoring Code
```java
Runtime runtime = Runtime.getRuntime();
long availableBytes = runtime.freeMemory();
System.out.println("Available memory: " + availableBytes / 1024 + " KB");
System.gc();  // Suggest garbage collection
```

### Defensive Copying Pattern
```java
// BAD: Escaping reference
public Map<String, Customer> getCustomers() {
    return this.records;
}

// GOOD: Unmodifiable view
public Map<String, Customer> getCustomers() {
    return Collections.unmodifiableMap(this.records);
}

// ALSO GOOD: Defensive copy
public Map<String, Customer> getCustomers() {
    return new HashMap<>(this.records);
}
```

### JPA Fetch Strategy
```java
@ManyToOne(fetch = FetchType.LAZY)  // Load on demand
private Tutor tutor;

@BatchSize(size = 10)  // Batch fetching optimization
@OneToMany(mappedBy = "tutor")
private List<Student> students;
```

---

## File Locations Quick Reference

### Configuration Files
- Spring app context: `StudentManager/src/application.xml`
- Spring MVC: `StudentManager/src/Dispatcher-servlet.xml`
- JPA config: `StudentManager/src/META-INF/persistence.xml`
- Logging: `StudentManager/src/log4j.properties`
- Hibernate queries: `StudentManager/src/META-INF/queries.xml`
- Web app: `StudentManager/web.xml`
- Build: `StudentManager/build.xml`

### Source Locations
- Domain model: `StudentManager/src/com/virtualpairprogrammers/domain/`
- DAOs: `StudentManager/src/com/virtualpairprogrammers/dao/`
- Services: `StudentManager/src/com/virtualpairprogrammers/services/`
- Controllers: `StudentManager/src/com/virtualpairprogrammers/controllers/`
- Web pages: `StudentManager/web/`

### Build Outputs
- Compiled classes: `*/build/` or `*/bin/`
- WAR file: Generated by Ant, deployed to `../tomcat/webapps/`

---

## Troubleshooting

### Build Issues
- **Missing libs:** Check `StudentManager/lib/` has all JARs
- **Ant not found:** Install Apache Ant or use IntelliJ
- **Compile errors:** Ensure Java version compatibility (pre-Java 8)

### Runtime Issues
- **Tomcat won't start:** Check port 8080 not in use
- **Out of Memory:** Increase heap: `-Xmx512M`
- **PermGen errors:** Use `-XX:MaxPermSize=256M`
- **ClassNotFound:** Check WAR deployed to `tomcat/webapps/`

### Database Issues
- **Note:** Database is intentionally disabled in `application.xml`
- Most operations use stub implementations
- If re-enabling DB, uncomment DataSource bean and configure Derby

---

## Version Information

- **Spring:** 3.0.3.RELEASE
- **Hibernate:** 4.1.4.Final
- **JPA:** 2.0
- **Servlet API:** 2.5
- **Tomcat:** Included version (6.x or 7.x era)
- **JMeter:** 2.12
- **Java:** Pre-Java 8 (no lambdas/streams in codebase)

---

*Last Updated: 2025-11-13*
*This is a living document. Update when making significant changes to the repository structure or adding new modules.*
