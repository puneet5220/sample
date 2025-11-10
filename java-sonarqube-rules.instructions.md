---
applyTo: "**/*.java"
description: "Comprehensive SonarSource Java code quality rules for code review and static analysis"
---

# SonarSource Java Code Quality Rules

**Version:** 1.0.0  
**Last Updated:** November 10, 2025  
**Source:** https://rules.sonarsource.com/java/  
**Total Rules:** 733 rules (60 Vulnerabilities, 175 Bugs, 40 Security Hotspots, 458 Code Smells)

## Overview

This document contains comprehensive Java code quality rules from SonarSource. These rules are used by SonarQube, SonarCloud, and SonarLint to identify bugs, vulnerabilities, security hotspots, and code smells in Java codebases.

## Rule Categories

### 1. Vulnerabilities (60 rules)
Security issues that could be exploited by attackers.

### 2. Bugs (175 rules)  
Code defects that will likely cause incorrect behavior or runtime errors.

### 3. Security Hotspots (40 rules)
Security-sensitive code that requires manual review.

### 4. Code Smells (458 rules)
Maintainability issues that make code harder to understand or modify.

---

## Critical Security Rules (BLOCKER/CRITICAL)

### S2068: Hard-coded passwords are security-sensitive
**Type:** Security Hotspot  
**Severity:** Critical

Hard-coded credentials in source code can be extracted by anyone with access to the code.

**Bad Example:**
```java
String password = "MySecretPassword123";  // Noncompliant
Connection conn = DriverManager.getConnection(url, "admin", "admin123");  // Noncompliant
```

**Good Example:**
```java
String password = System.getenv("DB_PASSWORD");  // Compliant
Properties props = loadFromSecureConfig();
Connection conn = DriverManager.getConnection(url, props);
```

---

### S6437: Credentials should not be hard-coded
**Type:** Vulnerability  
**Severity:** Blocker

Similar to S2068 but specifically targets hard-coded credentials.

---

### S2076: OS commands should not be vulnerable to command injection attacks
**Type:** Vulnerability  
**Severity:** Critical

User-controlled input should never be directly used in OS commands.

**Bad Example:**
```java
String input = request.getParameter("file");
Runtime.runtime().exec("cat " + input);  // Noncompliant - command injection
```

**Good Example:**
```java
String input = request.getParameter("file");
// Validate and sanitize input
if (input.matches("[a-zA-Z0-9_.-]+")) {
    ProcessBuilder pb = new ProcessBuilder("cat", input);  // Use ProcessBuilder with separate args
    pb.start();
}
```

---

### S2091: XPath expressions should not be vulnerable to injection attacks
**Type:** Vulnerability  
**Severity:** Critical

**Bad Example:**
```java
String userName = request.getParameter("user");
XPathExpression expr = xpath.compile("/users/user[@name='" + userName + "']");  // Noncompliant
```

**Good Example:**
```java
// Use parameterized XPath or sanitize input
String userName = sanitize(request.getParameter("user"));
```

---

### S3649: Database queries should not be vulnerable to injection attacks
**Type:** Vulnerability  
**Severity:** Critical

**Bad Example:**
```java
String query = "SELECT * FROM users WHERE name = '" + userName + "'";  // Noncompliant
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

**Good Example:**
```java
String query = "SELECT * FROM users WHERE name = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, userName);
ResultSet rs = stmt.executeQuery();
```

---

### S5131: Endpoints should not be vulnerable to reflected cross-site scripting (XSS) attacks
**Type:** Vulnerability  
**Severity:** Critical

**Bad Example:**
```java
@GetMapping("/welcome")
public String welcome(@RequestParam String name) {
    return "<html><body>Welcome " + name + "</body></html>";  // Noncompliant
}
```

**Good Example:**
```java
@GetMapping("/welcome")
public String welcome(@RequestParam String name, Model model) {
    model.addAttribute("name", name);  // Framework will escape
    return "welcome";  // Use template engine
}
```

---

### S4830: Server certificates should be verified during SSL/TLS connections
**Type:** Vulnerability  
**Severity:** Critical

**Bad Example:**
```java
SSLContext sslContext = SSLContext.getInstance("TLS");
sslContext.init(null, new TrustManager[] { new X509TrustManager() {
    public void checkClientTrusted(X509Certificate[] chain, String authType) {}
    public void checkServerTrusted(X509Certificate[] chain, String authType) {}  // Noncompliant
    public X509Certificate[] getAcceptedIssuers() { return null; }
}}, null);
```

**Good Example:**
```java
SSLContext sslContext = SSLContext.getInstance("TLS");
// Use default trust manager that validates certificates
sslContext.init(null, null, null);
```

---

## Critical Bug Rules

### S2259: Null pointers should not be dereferenced
**Type:** Bug  
**Severity:** Critical

**Bad Example:**
```java
String str = null;
if (condition) {
    str = "value";
}
int length = str.length();  // Noncompliant - may throw NullPointerException
```

**Good Example:**
```java
String str = null;
if (condition) {
    str = "value";
}
if (str != null) {
    int length = str.length();  // Compliant
}
```

---

### S2095: Resources should be closed
**Type:** Bug  
**Severity:** Blocker

**Bad Example:**
```java
FileInputStream fis = new FileInputStream(file);  // Noncompliant - not closed
byte[] data = new byte[1024];
fis.read(data);
```

**Good Example:**
```java
try (FileInputStream fis = new FileInputStream(file)) {
    byte[] data = new byte[1024];
    fis.read(data);
}  // Automatically closed
```

---

### S2093: Try-with-resources should be used
**Type:** Code Smell  
**Severity:** Major

Prefer try-with-resources over manual resource management.

**Bad Example:**
```java
FileReader fr = null;
try {
    fr = new FileReader(file);
    // use fr
} finally {
    if (fr != null) {
        fr.close();  // Noncompliant - use try-with-resources
    }
}
```

**Good Example:**
```java
try (FileReader fr = new FileReader(file)) {
    // use fr
}  // Automatically closed
```

---

### S1206: "equals(Object obj)" and "hashCode()" should be overridden in pairs
**Type:** Bug  
**Severity:** Critical

**Bad Example:**
```java
public class Person {
    private String name;
    
    @Override
    public boolean equals(Object obj) {  // Noncompliant - hashCode not overridden
        if (!(obj instanceof Person)) return false;
        return name.equals(((Person) obj).name);
    }
}
```

**Good Example:**
```java
public class Person {
    private String name;
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Person)) return false;
        return name.equals(((Person) obj).name);
    }
    
    @Override
    public int hashCode() {
        return name.hashCode();
    }
}
```

---

## Code Quality Best Practices

### S1192: String literals should not be duplicated
**Type:** Code Smell  
**Severity:** Minor

**Bad Example:**
```java
public void method() {
    System.out.println("Welcome to the application");
    logger.info("Welcome to the application");
    sendEmail("Welcome to the application");  // Noncompliant - duplicated 3 times
}
```

**Good Example:**
```java
private static final String WELCOME_MESSAGE = "Welcome to the application";

public void method() {
    System.out.println(WELCOME_MESSAGE);
    logger.info(WELCOME_MESSAGE);
    sendEmail(WELCOME_MESSAGE);
}
```

---

### S1118: Utility classes should not have public constructors
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
public class StringUtils {  // Noncompliant - has public constructor
    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }
}
```

**Good Example:**
```java
public class StringUtils {
    private StringUtils() {  // Private constructor
        throw new IllegalStateException("Utility class");
    }
    
    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }
}
```

---

### S1068: Unused "private" fields should be removed
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
public class MyClass {
    private int unusedField;  // Noncompliant - never used
    private String name;
    
    public String getName() {
        return name;
    }
}
```

**Good Example:**
```java
public class MyClass {
    private String name;
    
    public String getName() {
        return name;
    }
}
```

---

### S1144: Unused "private" methods should be removed
**Type:** Code Smell  
**Severity:** Major

---

### S1481: Unused local variables should be removed
**Type:** Code Smell  
**Severity:** Minor

**Bad Example:**
```java
public void method() {
    int unusedVar = 10;  // Noncompliant
    String name = "John";
    System.out.println(name);
}
```

---

### S1172: Unused method parameters should be removed
**Type:** Code Smell  
**Severity:** Major

**Exception:** Override methods, main methods, serialization methods

---

### S1854: Unused assignments should be removed
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
public int compute() {
    int result = 10;  // Noncompliant - overwritten before use
    result = calculate();
    return result;
}
```

---

## Naming Conventions

### S100: Method names should comply with a naming convention
**Type:** Code Smell  
**Severity:** Minor  
**Convention:** lowerCamelCase

**Bad Example:**
```java
public void MyMethod() {}  // Noncompliant
public void my_method() {}  // Noncompliant
```

**Good Example:**
```java
public void myMethod() {}  // Compliant
```

---

### S101: Class names should comply with a naming convention
**Type:** Code Smell  
**Severity:** Minor  
**Convention:** UpperCamelCase

---

### S115: Constant names should comply with a naming convention
**Type:** Code Smell  
**Severity:** Minor  
**Convention:** UPPER_SNAKE_CASE

**Bad Example:**
```java
public static final String maxValue = "100";  // Noncompliant
```

**Good Example:**
```java
public static final String MAX_VALUE = "100";  // Compliant
```

---

### S116: Field names should comply with a naming convention
**Type:** Code Smell  
**Severity:** Minor  
**Convention:** lowerCamelCase

---

### S117: Local variable and method parameter names should comply with a naming convention
**Type:** Code Smell  
**Severity:** Minor  
**Convention:** lowerCamelCase

---

## Exception Handling

### S1181: Throwable and Error should not be caught
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
try {
    doSomething();
} catch (Throwable t) {  // Noncompliant - too broad
    log(t);
}
```

**Good Example:**
```java
try {
    doSomething();
} catch (SpecificException e) {  // Catch specific exceptions
    log(e);
}
```

---

### S1166: Exception handlers should preserve the original exceptions
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
try {
    doSomething();
} catch (IOException e) {
    throw new MyException("Error occurred");  // Noncompliant - lost original exception
}
```

**Good Example:**
```java
try {
    doSomething();
} catch (IOException e) {
    throw new MyException("Error occurred", e);  // Preserve cause
}
```

---

### S2142: "InterruptedException" and "ThreadDeath" should not be ignored
**Type:** Bug  
**Severity:** Critical

**Bad Example:**
```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {  // Noncompliant - ignored
    // Do nothing
}
```

**Good Example:**
```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();  // Restore interrupted status
    log.error("Thread interrupted", e);
}
```

---

## Thread Safety

### S2445: Blocks should be synchronized on "private final" fields
**Type:** Bug  
**Severity:** Critical

**Bad Example:**
```java
public class MyClass {
    private Object lock = new Object();  // Noncompliant - not final
    
    public void method() {
        synchronized(lock) {
            // ...
        }
    }
}
```

**Good Example:**
```java
public class MyClass {
    private final Object lock = new Object();  // Compliant
    
    public void method() {
        synchronized(lock) {
            // ...
        }
    }
}
```

---

### S2885: Non-thread-safe fields should not be static
**Type:** Bug  
**Severity:** Critical

**Bad Example:**
```java
public class DateFormatter {
    private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");  // Noncompliant - not thread-safe
}
```

**Good Example:**
```java
public class DateFormatter {
    private static final DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");  // Thread-safe
    
    // Or use ThreadLocal
    private static ThreadLocal<SimpleDateFormat> sdf = ThreadLocal.withInitial(
        () -> new SimpleDateFormat("yyyy-MM-dd")
    );
}
```

---

## Collections

### S2175: Inappropriate "Collection" calls should not be made
**Type:** Bug  
**Severity:** Critical

**Bad Example:**
```java
List<String> strings = new ArrayList<>();
Integer number = 42;
strings.remove(number);  // Noncompliant - wrong type, removes by index instead
```

---

### S2864: "entrySet()" should be iterated when both the key and value are needed
**Type:** Code Smell  
**Severity:** Minor

**Bad Example:**
```java
Map<String, Integer> map = new HashMap<>();
for (String key : map.keySet()) {  // Noncompliant
    Integer value = map.get(key);
    process(key, value);
}
```

**Good Example:**
```java
Map<String, Integer> map = new HashMap<>();
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    process(entry.getKey(), entry.getValue());
}
```

---

### S1155: "Collection.isEmpty()" should be used to test for emptiness
**Type:** Code Smell  
**Severity:** Minor

**Bad Example:**
```java
if (collection.size() == 0) {  // Noncompliant
    // ...
}
```

**Good Example:**
```java
if (collection.isEmpty()) {  // Compliant
    // ...
}
```

---

## Modern Java Features

### S6204: "Stream.toList()" method should be used instead of "collectors"
**Type:** Code Smell  
**Severity:** Minor  
**Since:** Java 16

**Bad Example:**
```java
List<String> list = stream.collect(Collectors.toList());  // Noncompliant
```

**Good Example:**
```java
List<String> list = stream.toList();  // Compliant (Java 16+)
```

---

### S6212: Local-Variable Type Inference should be used
**Type:** Code Smell  
**Severity:** Minor  
**Since:** Java 10

**Bad Example:**
```java
ArrayList<String> list = new ArrayList<>();  // Noncompliant
```

**Good Example:**
```java
var list = new ArrayList<String>();  // Compliant (Java 10+)
```

---

### S6201: Pattern Matching for "instanceof" operator should be used
**Type:** Code Smell  
**Severity:** Minor  
**Since:** Java 16

**Bad Example:**
```java
if (obj instanceof String) {
    String str = (String) obj;  // Noncompliant
    process(str);
}
```

**Good Example:**
```java
if (obj instanceof String str) {  // Compliant (Java 16+)
    process(str);
}
```

---

### S1604: Anonymous inner classes containing only one method should become lambdas
**Type:** Code Smell  
**Severity:** Major  
**Since:** Java 8

**Bad Example:**
```java
Runnable r = new Runnable() {  // Noncompliant
    @Override
    public void run() {
        doSomething();
    }
};
```

**Good Example:**
```java
Runnable r = () -> doSomething();  // Compliant
```

---

### S1612: Lambdas should be replaced with method references
**Type:** Code Smell  
**Severity:** Minor

**Bad Example:**
```java
list.forEach(x -> System.out.println(x));  // Noncompliant
```

**Good Example:**
```java
list.forEach(System.out::println);  // Compliant
```

---

## Android-Specific Rules

### S6363: Enabling file access for WebViews is security-sensitive
**Type:** Security Hotspot

**Bad Example:**
```java
WebView webView = new WebView(context);
webView.getSettings().setAllowFileAccess(true);  // Security-sensitive
```

---

### S6362: Enabling JavaScript support for WebViews is security-sensitive
**Type:** Security Hotspot

**Bad Example:**
```java
WebView webView = new WebView(context);
webView.getSettings().setJavaScriptEnabled(true);  // Security-sensitive
```

---

### S6301: Mobile database encryption keys should not be disclosed
**Type:** Vulnerability

**Bad Example:**
```java
String key = "MyEncryptionKey123";  // Noncompliant
SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(file, key, null);
```

---

## Spring Framework Rules

### S4288: Spring components should use constructor injection
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // Noncompliant - field injection
}
```

**Good Example:**
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired  // Optional in Spring 4.3+
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

---

### S6813: Field dependency injection should be avoided
**Type:** Code Smell

Use constructor injection for mandatory dependencies.

---

### S6809: Methods with Spring proxy should not be called via "this"
**Type:** Code Smell

**Bad Example:**
```java
@Service
public class MyService {
    @Transactional
    public void methodA() { }
    
    public void methodB() {
        this.methodA();  // Noncompliant - proxy not applied
    }
}
```

**Good Example:**
```java
@Service
public class MyService {
    @Autowired
    private MyService self;  // Inject self to get proxy
    
    @Transactional
    public void methodA() { }
    
    public void methodB() {
        self.methodA();  // Compliant - proxy applied
    }
}
```

---

## Complexity Rules

### S3776: Cognitive Complexity of methods should not be too high
**Type:** Code Smell  
**Severity:** Critical  
**Default Threshold:** 15

Cognitive Complexity measures how difficult code is to understand.

---

### S1541: Methods should not be too complex
**Type:** Code Smell  
**Severity:** Critical  
**Default Threshold:** 10 (Cyclomatic Complexity)

---

### S1067: Expressions should not be too complex
**Type:** Code Smell  
**Default Threshold:** 3

**Bad Example:**
```java
if (a && b || c && d || e && f) {  // Noncompliant - too complex
    // ...
}
```

**Good Example:**
```java
boolean condition1 = a && b;
boolean condition2 = c && d;
boolean condition3 = e && f;
if (condition1 || condition2 || condition3) {  // Compliant
    // ...
}
```

---

## Documentation

### S1176: Public types, methods and fields (API) should be documented with Javadoc
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
public class UserService {  // Noncompliant - no Javadoc
    public User findUser(String id) {  // Noncompliant
        // ...
    }
}
```

**Good Example:**
```java
/**
 * Service for managing user operations.
 */
public class UserService {
    /**
     * Finds a user by ID.
     * @param id the user ID
     * @return the user, or null if not found
     */
    public User findUser(String id) {
        // ...
    }
}
```

---

## Performance

### S1643: Strings should not be concatenated using '+' in a loop
**Type:** Code Smell  
**Severity:** Major

**Bad Example:**
```java
String result = "";
for (String s : list) {
    result += s;  // Noncompliant - creates new String each iteration
}
```

**Good Example:**
```java
StringBuilder result = new StringBuilder();
for (String s : list) {
    result.append(s);
}
String finalResult = result.toString();
```

---

### S2140: Methods of "Random" that return floating point values should not be used in random integer generation
**Type:** Code Smell

**Bad Example:**
```java
int randomInt = (int) (Math.random() * 100);  // Noncompliant
```

**Good Example:**
```java
Random random = new Random();
int randomInt = random.nextInt(100);  // Compliant
```

---

## Code Style

### S1128: Unnecessary imports should be removed
**Type:** Code Smell  
**Severity:** Minor

---

### S2208: Wildcard imports should not be used
**Type:** Code Smell  
**Severity:** Minor

**Bad Example:**
```java
import java.util.*;  // Noncompliant
```

**Good Example:**
```java
import java.util.ArrayList;
import java.util.HashMap;
```

---

### S1123: Deprecated elements should have both the annotation and the Javadoc tag
**Type:** Code Smell

**Bad Example:**
```java
@Deprecated
public void oldMethod() {}  // Noncompliant - missing @deprecated Javadoc
```

**Good Example:**
```java
/**
 * @deprecated Use {@link #newMethod()} instead
 */
@Deprecated
public void oldMethod() {}
```

---

### S1133: Deprecated code should be removed
**Type:** Code Smell  
**Severity:** Info

Remove code that has been deprecated for a long time.

---

## Testing Best Practices

### S2699: Tests should include assertions
**Type:** Code Smell  
**Severity:** Critical

**Bad Example:**
```java
@Test
public void testUserCreation() {
    User user = new User("John");  // Noncompliant - no assertion
}
```

**Good Example:**
```java
@Test
public void testUserCreation() {
    User user = new User("John");
    assertNotNull(user);
    assertEquals("John", user.getName());
}
```

---

### S5786: JUnit5 test classes and methods should have default package visibility
**Type:** Code Smell

**Bad Example:**
```java
public class UserServiceTest {  // Noncompliant - should be package-private
    @Test
    public void testFindUser() {  // Noncompliant
        // ...
    }
}
```

**Good Example:**
```java
class UserServiceTest {  // Compliant (JUnit 5)
    @Test
    void testFindUser() {
        // ...
    }
}
```

---

### S5976: Similar tests should be grouped in a single Parameterized test
**Type:** Code Smell

**Bad Example:**
```java
@Test
void testAddition1() { assertEquals(2, calculator.add(1, 1)); }

@Test
void testAddition2() { assertEquals(4, calculator.add(2, 2)); }

@Test
void testAddition3() { assertEquals(6, calculator.add(3, 3)); }
```

**Good Example:**
```java
@ParameterizedTest
@CsvSource({ "1,1,2", "2,2,4", "3,3,6" })
void testAddition(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

---

## Summary Statistics

**Total Rules:** 733
- **Vulnerabilities:** 60
- **Bugs:** 175  
- **Security Hotspots:** 40
- **Code Smells:** 458

**Quick Fix Available:** 65 rules

**Rule Severity Distribution:**
- **Blocker:** ~15 rules
- **Critical:** ~80 rules
- **Major:** ~350 rules
- **Minor:** ~250 rules
- **Info:** ~38 rules

---

## References

- **SonarSource Rules Documentation:** https://rules.sonarsource.com/java/
- **SonarQube Documentation:** https://docs.sonarqube.org/
- **SonarLint IDE Plugin:** https://www.sonarsource.com/products/sonarlint/
- **Clean Code Principles:** Robert C. Martin
- **Effective Java:** Joshua Bloch

---

## Tool Integration

These rules can be enforced through:

1. **SonarQube Server** (Self-hosted)
2. **SonarCloud** (SaaS)
3. **SonarLint** (IDE integration - IntelliJ, Eclipse, VS Code, Visual Studio)
4. **CI/CD Pipeline** Integration (Jenkins, Azure DevOps, GitHub Actions)

---

## Quick Reference Checklist

### Before Committing Code:

- [ ] No hard-coded credentials or secrets
- [ ] All resources closed properly (try-with-resources)
- [ ] No SQL injection vulnerabilities (use PreparedStatement)
- [ ] No null pointer dereferences
- [ ] equals() and hashCode() overridden together
- [ ] Proper exception handling (don't swallow exceptions)
- [ ] No unused variables, fields, or methods
- [ ] Follow naming conventions
- [ ] Public API has Javadoc
- [ ] Tests include assertions
- [ ] No code duplication (DRY principle)
- [ ] Thread-safe if multi-threaded
- [ ] Use modern Java features (var, pattern matching, streams)
- [ ] Cognitive complexity under 15

---

**Note:** This instruction file contains the most critical and commonly applicable rules. For the complete list of all 733 rules with detailed descriptions, examples, and rationale, refer to the official SonarSource documentation at https://rules.sonarsource.com/java/
