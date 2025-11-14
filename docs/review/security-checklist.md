# Security Checklist for Pull Request Reviews

This checklist provides comprehensive security validation rules for code reviews. All items should be verified during PR review to prevent security vulnerabilities.

---

## 1. Secrets and Credentials Management {#secrets-management}

### No Secrets in Code
- [ ] No hardcoded passwords, API keys, tokens, or credentials in source code
- [ ] No credentials in configuration files committed to repository
- [ ] No secrets in test files or test data
- [ ] No credentials in comments or documentation

### Proper Secret Storage
- [ ] Use Key Vault, AWS Secrets Manager, or equivalent secure storage
- [ ] Secrets loaded via environment variables or secure configuration providers
- [ ] Required environment variables documented in README or setup guide
- [ ] No secrets in logs or error messages

**Examples of violations:**

**React/TypeScript:**
```typescript
// ❌ BAD
const API_KEY = "sk-1234567890abcdef";
const dbConfig = {
  password: "hardcoded-password"
};

// ✅ GOOD
const API_KEY = process.env.REACT_APP_API_KEY;
const dbConfig = {
  password: process.env.DB_PASSWORD
};
```

**Kotlin:**
```kotlin
// ❌ BAD
val apiKey = "sk-1234567890abcdef"
val dbPassword = "hardcoded-password-123"

// ✅ GOOD
val apiKey = System.getenv("API_KEY")
val dbPassword = secretsManager.getSecret("DB_PASSWORD")
```

---

## 2. Input and Output Validation {#input-output-validation}

### XSS (Cross-Site Scripting) Prevention
- [ ] All user input is properly escaped before rendering in HTML
- [ ] Use framework-provided escaping mechanisms (React's JSX, template engines)
- [ ] Sanitize rich text input with approved libraries (DOMPurify, etc.)
- [ ] Set proper Content-Security-Policy headers

### SQL Injection Prevention
- [ ] Use parameterized queries or prepared statements exclusively
- [ ] No string concatenation for building SQL queries
- [ ] ORM/query builders used correctly (not bypassing parameterization)
- [ ] Input validation for column names and table names (when dynamic)

**Kotlin Example:**
```kotlin
// ❌ BAD - SQL Injection vulnerable
val query = "SELECT * FROM users WHERE id = $userId"

// ✅ GOOD - Parameterized query
val query = "SELECT * FROM users WHERE id = ?"
val stmt = connection.prepareStatement(query)
stmt.setInt(1, userId)
```

**React/TypeScript Example:**
```typescript
// ❌ BAD - SQL Injection in API call
const response = await fetch(`/api/users?query=SELECT * FROM users WHERE name='${userName}'`);

// ✅ GOOD - Use proper API with validation
const response = await fetch('/api/users', {
  method: 'POST',
  body: JSON.stringify({ userName })
});
```

### NoSQL Injection Prevention
- [ ] MongoDB/DynamoDB queries use proper operators, not string concatenation
- [ ] Input validation for query operators
- [ ] No direct use of user input in query construction

### SSRF (Server-Side Request Forgery) Prevention
- [ ] Validate and whitelist URLs before making server-side requests
- [ ] No direct use of user-provided URLs in HTTP clients
- [ ] Implement URL scheme restrictions (allow only https://)
- [ ] Validate and restrict IP ranges (block private IPs: 127.0.0.1, 10.x.x.x, etc.)

**React/TypeScript Example:**
```typescript
// ❌ BAD - SSRF vulnerable
const url = req.body.webhookUrl;
await axios.get(url);

// ✅ GOOD - Validated URL
const url = req.body.webhookUrl;
if (!isAllowedDomain(url)) throw new Error('Invalid domain');
await axios.get(url);
```

**Kotlin Example:**
```kotlin
// ❌ BAD - SSRF vulnerable
val url = request.getParameter("imageUrl")
val image = URL(url).readBytes()

// ✅ GOOD - Validated URL
val url = request.getParameter("imageUrl")
require(isAllowedDomain(url)) { "Invalid domain" }
val image = URL(url).readBytes()
```

---

## 3. Authentication and Authorization {#auth-management}

### Token Management
- [ ] Tokens have appropriate expiration times (not excessive)
- [ ] Refresh token rotation implemented
- [ ] Token revocation mechanism exists
- [ ] Tokens use sufficient entropy/randomness

### Access Control
- [ ] Proper authorization checks before resource access
- [ ] Principle of least privilege applied (minimal scopes/permissions)
- [ ] No authorization checks bypassed or commented out
- [ ] Role-based or attribute-based access control properly implemented

### Session Management
- [ ] Sessions expire after inactivity
- [ ] Secure session storage (httpOnly, secure, sameSite cookies)
- [ ] Session fixation prevention
- [ ] Logout properly invalidates sessions

**React/TypeScript Example:**
```typescript
// ❌ BAD - No authorization check
app.get('/api/users/:id', async (req, res) => {
  const user = await db.getUser(req.params.id);
  res.json(user);
});

// ✅ GOOD - Proper authorization
app.get('/api/users/:id', authenticate, async (req, res) => {
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  const user = await db.getUser(req.params.id);
  res.json(user);
});
```

**Kotlin Example:**
```kotlin
// ❌ BAD - No authorization check
@GetMapping("/api/users/{id}")
fun getUser(@PathVariable id: String): User {
    return userRepository.findById(id)
}

// ✅ GOOD - Proper authorization
@GetMapping("/api/users/{id}")
fun getUser(@PathVariable id: String, @AuthenticationPrincipal user: User): User {
    require(user.id == id || user.isAdmin) { "Forbidden" }
    return userRepository.findById(id)
}
```

---

## 4. Dependency Vulnerabilities {#dependency-security}

### Known CVEs
- [ ] No dependencies with known critical or high-severity CVEs
- [ ] Dependencies updated to latest secure versions
- [ ] Vulnerability scanning enabled in CI/CD pipeline
- [ ] SBOM (Software Bill of Materials) generated and reviewed

### License Compliance
- [ ] No dependencies with incompatible licenses
- [ ] License compatibility verified for commercial use
- [ ] Copyleft licenses properly handled

**Tools to use:**
- `npm audit` / `yarn audit` for Node.js
- `pip-audit` for Python
- OWASP Dependency-Check
- Snyk, GitHub Dependabot

---

## 5. Encryption {#encryption}

### Data in Transit
- [ ] All external communications use TLS 1.2 or higher
- [ ] No HTTP connections to external services (use HTTPS)
- [ ] Certificate validation not disabled
- [ ] No hardcoded trust of self-signed certificates in production

### Data at Rest
- [ ] Sensitive data encrypted at rest when applicable
- [ ] Use industry-standard encryption algorithms (AES-256, etc.)
- [ ] Encryption keys properly managed (not hardcoded)
- [ ] Database encryption enabled for sensitive data

**React/TypeScript Example:**
```typescript
// ❌ BAD - Insecure connection
const response = await fetch('http://api.example.com/data', { 
  // @ts-ignore
  rejectUnauthorized: false 
});

// ✅ GOOD - Secure connection
const response = await fetch('https://api.example.com/data');
```

**Kotlin Example:**
```kotlin
// ❌ BAD - Insecure connection
val client = OkHttpClient.Builder()
    .hostnameVerifier { _, _ -> true }
    .build()

// ✅ GOOD - Secure connection
val client = OkHttpClient.Builder()
    .build()
val response = client.newCall(
    Request.Builder()
        .url("https://api.example.com/data")
        .build()
).execute()
```

---

## 6. PII and Sensitive Data Handling {#pii-handling}

### Logging and Monitoring
- [ ] No PII (Personally Identifiable Information) in logs
- [ ] Sensitive data masked in logs (credit cards, SSN, passwords)
- [ ] No sensitive data in error messages shown to users
- [ ] Audit logs for sensitive operations

### Data Minimization
- [ ] Only collect necessary PII
- [ ] PII retention policies implemented
- [ ] Data anonymization for analytics/testing

**React/TypeScript Example:**
```typescript
// ❌ BAD - PII in logs
logger.info(`User login: ${user.email}, SSN: ${user.ssn}`);

// ✅ GOOD - Masked PII
logger.info(`User login: ${maskEmail(user.email)}`);
```

**Kotlin Example:**
```kotlin
// ❌ BAD - PII in logs
logger.info("User login: ${user.email}, SSN: ${user.ssn}")

// ✅ GOOD - Masked PII
logger.info("User login: ${maskEmail(user.email)}")
```

---

## 7. Additional Security Considerations

### CORS (Cross-Origin Resource Sharing)
- [ ] CORS properly configured (not wildcard `*` in production)
- [ ] Allowed origins explicitly whitelisted
- [ ] Credentials handling properly configured

### Rate Limiting and DoS Prevention
- [ ] Rate limiting implemented on public endpoints
- [ ] Request size limits enforced
- [ ] Timeout configurations prevent resource exhaustion

### File Upload Security
- [ ] File type validation (whitelist approach)
- [ ] File size limits enforced
- [ ] Uploaded files scanned for malware
- [ ] Files stored outside web root or with proper access controls

---

## Review Checklist Summary

Use this quick checklist during PR reviews:

- [ ] **Secrets**: No hardcoded credentials, using secure storage
- [ ] **Input Validation**: Parameterized queries, escaped output, SSRF prevention
- [ ] **Auth/AuthZ**: Proper authentication and authorization checks
- [ ] **Dependencies**: No known CVEs, licenses compatible
- [ ] **Encryption**: TLS for transit, encryption for sensitive data at rest
- [ ] **PII**: Masked in logs, data minimization applied
- [ ] **Additional**: CORS, rate limiting, file upload security

---

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
