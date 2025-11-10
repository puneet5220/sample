---
description: "Prompt for reviewing Java files against SonarSource code quality rules"
---

# Java Code Review Prompt

**Version:** 1.0.0  
**Last Updated:** November 10, 2025  
**Instruction File:** `.github/instructions/java-sonarqube-rules.instructions.md`

## Purpose

This prompt template is used to review any given Java file and highlight issues found based on SonarSource Java code quality rules. The review covers vulnerabilities, bugs, security hotspots, and code smells defined in the instruction file.

---

## Prompt Template

```markdown
# Java Code Review Request

## Task
Review the following Java file against SonarSource code quality rules and identify all issues.

## File Information
- **File Path:** [Provide file path]
- **File Name:** [Provide file name]
- **Purpose:** [Brief description of what the file does]

## Review Scope
Analyze the file for:
1. **Vulnerabilities** (Security issues that could be exploited)
2. **Bugs** (Code defects causing incorrect behavior)
3. **Security Hotspots** (Security-sensitive code requiring review)
4. **Code Smells** (Maintainability issues)

## Code to Review
```java
[Paste Java code here]
```

## Required Analysis

### 1. Critical Security Issues (BLOCKER/CRITICAL)
Check for:
- [ ] S2068/S6437: Hard-coded credentials or passwords
- [ ] S2076: Command injection vulnerabilities
- [ ] S2091: XPath injection vulnerabilities
- [ ] S3649: SQL injection vulnerabilities
- [ ] S5131: XSS (Cross-Site Scripting) vulnerabilities
- [ ] S4830: SSL/TLS certificate validation issues
- [ ] S6301: Mobile database encryption key disclosure (Android)
- [ ] S6363: WebView file access security (Android)
- [ ] S6362: WebView JavaScript security (Android)

### 2. Critical Bug Patterns
Check for:
- [ ] S2259: Null pointer dereferences
- [ ] S2095: Unclosed resources
- [ ] S2093: Missing try-with-resources
- [ ] S1206: equals()/hashCode() contract violation
- [ ] S2142: Ignored InterruptedException
- [ ] S2175: Inappropriate Collection calls
- [ ] S2445: Synchronization on non-final fields
- [ ] S2885: Non-thread-safe static fields

### 3. Code Quality Issues
Check for:
- [ ] S1192: Duplicated string literals
- [ ] S1118: Utility class with public constructor
- [ ] S1068: Unused private fields
- [ ] S1144: Unused private methods
- [ ] S1481: Unused local variables
- [ ] S1172: Unused method parameters
- [ ] S1854: Unused assignments

### 4. Naming Conventions
Check for:
- [ ] S100: Method names (lowerCamelCase)
- [ ] S101: Class names (UpperCamelCase)
- [ ] S115: Constant names (UPPER_SNAKE_CASE)
- [ ] S116: Field names (lowerCamelCase)
- [ ] S117: Local variable names (lowerCamelCase)

### 5. Exception Handling
Check for:
- [ ] S1181: Catching Throwable/Error
- [ ] S1166: Lost exception information
- [ ] S1163: Exceptions thrown in finally blocks
- [ ] S2221: Unnecessary Exception catching

### 6. Modern Java Features (if applicable)
Check for:
- [ ] S6204: Use Stream.toList() instead of Collectors.toList() (Java 16+)
- [ ] S6212: Use var for local variable type inference (Java 10+)
- [ ] S6201: Use pattern matching for instanceof (Java 16+)
- [ ] S1604: Convert anonymous classes to lambdas (Java 8+)
- [ ] S1612: Replace lambdas with method references

### 7. Android-Specific Rules (if applicable)
Check for:
- [ ] S6363: WebView file access security
- [ ] S6362: WebView JavaScript security
- [ ] S6301: Database encryption key handling
- [ ] S6293: Biometric authentication security
- [ ] S6291: Unencrypted database usage

### 8. Spring Framework Rules (if applicable)
Check for:
- [ ] S4288: Use constructor injection
- [ ] S6813: Avoid field injection
- [ ] S6809: Don't call @Transactional methods via "this"
- [ ] S6838: @Bean method invocation with proxyBeanMethods=false
- [ ] S6832: Non-singleton beans in singleton beans

### 9. Complexity Analysis
Check for:
- [ ] S3776: Cognitive Complexity > 15
- [ ] S1541: Cyclomatic Complexity > 10
- [ ] S1067: Expression complexity > 3

### 10. Documentation
Check for:
- [ ] S1176: Missing Javadoc on public API
- [ ] S1123: Deprecated elements missing @deprecated tag

---

## Output Format

Provide the review results in the following format:

### Summary
- **Total Issues Found:** [Number]
- **Critical Issues:** [Number]
- **Major Issues:** [Number]
- **Minor Issues:** [Number]

### Issues by Category

#### 🔴 CRITICAL - Vulnerabilities
[List all critical security vulnerabilities found]

**Example:**
```
Issue: S2068 - Hard-coded password
Location: Line 45
Code: `String password = "MySecret123";`
Severity: Critical
Impact: Credentials exposed in source code
Recommendation: Use environment variables or secure configuration
Fix:
```java
String password = System.getenv("DB_PASSWORD");
```
```

#### 🔴 CRITICAL - Bugs
[List all critical bugs found]

#### 🟡 MAJOR - Code Smells
[List all major code quality issues]

#### 🟢 MINOR - Code Style
[List all minor issues]

### Detailed Findings

For each issue found, provide:

1. **Rule ID and Name:** [e.g., S2259 - Null pointers should not be dereferenced]
2. **Severity:** [Blocker/Critical/Major/Minor/Info]
3. **Type:** [Vulnerability/Bug/Code Smell/Security Hotspot]
4. **Location:** [Line number(s)]
5. **Problematic Code:**
   ```java
   [Show the problematic code snippet]
   ```
6. **Explanation:** [Why this is an issue]
7. **Recommended Fix:**
   ```java
   [Show the corrected code]
   ```
8. **Impact:** [What could go wrong if not fixed]

### Code Quality Metrics

- **Cognitive Complexity:** [Score]
- **Cyclomatic Complexity:** [Score]
- **Duplicated Code Blocks:** [Number]
- **Test Coverage Estimate:** [Percentage if applicable]

### Priority Recommendations

List top 5 issues to fix immediately:

1. [Issue with highest priority]
2. [Second priority issue]
3. [Third priority issue]
4. [Fourth priority issue]
5. [Fifth priority issue]

### Positive Observations

[List good practices found in the code]

### Overall Assessment

**Code Quality Score:** [Excellent/Good/Fair/Poor]

**Summary:** [Brief overall assessment of code quality]

**Next Steps:** [Recommended actions]

---

## Example Usage

### Example 1: Review a Service Class

```markdown
# Java Code Review Request

## File Information
- **File Path:** src/main/java/com/example/UserService.java
- **File Name:** UserService.java
- **Purpose:** Service class for user management operations

## Code to Review
```java
package com.example;

import java.sql.*;

public class UserService {
    private String dbPassword = "admin123";  // Hard-coded password
    
    public User findUser(String userId) throws SQLException {
        Connection conn = null;
        Statement stmt = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost/db", "admin", dbPassword);
            stmt = conn.createStatement();
            String query = "SELECT * FROM users WHERE id = '" + userId + "'";  // SQL injection
            ResultSet rs = stmt.executeQuery(query);
            // ... process results
        } finally {
            // Resources not properly closed
        }
        return null;
    }
}
```
```

**Expected Output:**
```markdown
### Summary
- **Total Issues Found:** 4
- **Critical Issues:** 3
- **Major Issues:** 1
- **Minor Issues:** 0

### Issues by Category

#### 🔴 CRITICAL - Vulnerabilities

**Issue 1: S2068 - Hard-coded password**
- Location: Line 6
- Code: `private String dbPassword = "admin123";`
- Severity: Critical
- Impact: Database credentials exposed in source code
- Fix:
  ```java
  private String dbPassword = System.getenv("DB_PASSWORD");
  ```

**Issue 2: S3649 - SQL Injection**
- Location: Line 13
- Code: `String query = "SELECT * FROM users WHERE id = '" + userId + "'";`
- Severity: Critical
- Impact: User input directly concatenated into SQL query
- Fix:
  ```java
  String query = "SELECT * FROM users WHERE id = ?";
  PreparedStatement pstmt = conn.prepareStatement(query);
  pstmt.setString(1, userId);
  ResultSet rs = pstmt.executeQuery();
  ```

#### 🔴 CRITICAL - Bugs

**Issue 3: S2095 - Resources not closed**
- Location: Lines 10-15
- Severity: Blocker
- Impact: Connection and Statement leaks
- Fix:
  ```java
  try (Connection conn = DriverManager.getConnection(...);
       PreparedStatement stmt = conn.prepareStatement(query)) {
      // Use resources
  }  // Automatically closed
  ```

[Continue with remaining issues...]
```

---

## Advanced Usage

### Custom Rule Sets

To focus on specific rule categories, modify the checklist:

**Example: Security-Only Review**
```markdown
## Required Analysis

### Security Rules Only
- [ ] All S2xxx security vulnerability rules
- [ ] All S6xxx Android security rules
- [ ] All Sxxxx security hotspot rules
```

### Android-Specific Review
```markdown
## Required Analysis

### Android Security Review
- [ ] WebView security (S6362, S6363)
- [ ] Database encryption (S6301, S6291)
- [ ] Biometric authentication (S6293)
- [ ] File access security (S6300)
- [ ] Intent security (S6384, S5320, S5322)
```

### Spring Boot Review
```markdown
## Required Analysis

### Spring Framework Best Practices
- [ ] Dependency injection (S4288, S6813, S3306)
- [ ] Transaction management (S6809, S2229)
- [ ] Bean configuration (S6862, S6838, S6832)
- [ ] Request mapping (S4488, S3751, S6806)
```

---

## Integration with Development Workflow

### Pre-Commit Review
Use this prompt before committing code to catch issues early.

### Pull Request Review
Include code review results in PR descriptions for team awareness.

### CI/CD Integration
Automate reviews using SonarQube Scanner in your build pipeline.

### IDE Integration
Use SonarLint plugin in your IDE for real-time feedback.

---

## References

- **Instruction File:** `.github/instructions/java-sonarqube-rules.instructions.md`
- **SonarSource Rules:** https://rules.sonarsource.com/java/
- **SonarQube Documentation:** https://docs.sonarqube.org/

---

## Tips for Effective Reviews

1. **Focus on Critical Issues First:** Address vulnerabilities and bugs before code smells
2. **Provide Context:** Explain why each issue matters
3. **Suggest Specific Fixes:** Show exact code improvements
4. **Be Constructive:** Highlight positive patterns too
5. **Consider Project Context:** Some rules may not apply to all projects
6. **Use Automated Tools:** Complement manual review with SonarQube/SonarLint

---

## Version History

- **v1.0.0** (2025-11-10): Initial version with comprehensive SonarSource rules
