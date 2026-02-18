# Spring Security: Common Web Attacks - Notes

## Overview
- Understanding common attacks is essential to grasp the importance of Spring Security
- This session covers how attacks happen and protection mechanisms

---

## 1. CSRF (Cross-Site Request Forgery)

### What is CSRF?
- Tricks a browser into making unwanted requests to a site where the user is already authenticated
- Browser automatically adds session ID/cookies to requests

### How it Works:
1. User is authenticated to a website (e.g., gmail.com)
2. Session ID is stored in browser cookies
3. Attacker sends malicious link to user
4. User clicks the link unknowingly
5. Browser automatically includes session ID in the request to the authenticated site
6. Server validates the session and processes the unwanted request

### Applicable Scenarios:
- Session-based authentication (stateful)
- Where cookies store session information

### Protection:
**CSRF Token**
- Server returns a unique token in response
- Token known only to legitimate/authenticated sources
- Token must be appended to every request
- Attacker cannot forge requests without knowing the token

---

## 2. XSS (Cross-Site Scripting)

### What is XSS?
- Attacker injects malicious scripts into web pages viewed by other users
- Browser executes the script, compromising security

### How it Works:
1. Web page has comment section where users post content
2. Attacker posts comment containing malicious script (e.g., `<script>alert('XSS')</script>`)
3. Script is stored in database/memory
4. When other users view the page, script executes in their browser
5. Attacker can steal session cookies, deform website, or redirect to malicious sites

### Consequences:
- Session/cookie theft
- Unauthorized data access
- Website defacement

### Protection:
1. **Proper Input Escaping**
    - Convert special characters to safe equivalents
    - Browser treats input as string, not executable code

2. **Input Validation**
    - Define what values are allowed
    - Reject suspicious input

---

## 3. CORS (Cross-Origin Resource Sharing)

### What is CORS?
- Not an attack, but a security feature
- Restricts web pages from making requests to different origins unless allowed by server

### Origin Definition:
- Combination of: **Protocol + Domain + Port**
- All three must match for same origin

### Examples of Different Origins:
- `https://localhost:8080` vs `http://localhost:8080` (different protocol)
- `https://localhost:8080` vs `https://localhost:9090` (different port)
- `https://sub.localhost` vs `https://localhost` (different domain)

### Protection:
- Server explicitly whitelists allowed origins
- Use `Access-Control-Allow-Origin` header
- Define allowed methods (GET, POST, PUT, DELETE)
- Define allowed headers

### Role in Security:
- Acts as first line of defense against CSRF attacks
- Prevents requests from unauthorized origins even before CSRF token validation

---

## 4. SQL Injection

### What is SQL Injection?
- Attacker manipulates SQL queries by inserting malicious input into user fields
- Directly using user input in SQL queries without validation

### How it Works:
1. Simple query: `SELECT * FROM products WHERE type = '<user_input>'`
2. User inputs: `' OR '1'='1`
3. Query becomes: `SELECT * FROM products WHERE type = '' OR '1'='1'`
4. Returns all data instead of filtered results

### Consequences:
- Unauthorized data access
- Data deletion or modification
- Database table/column discovery
- Complete database compromise

### Protection:
**Parameterized Queries (Prepared Statements)**
```
Query: "SELECT * FROM user_details WHERE username = ?"
Set parameter: username = user_input
```
- User input treated as value, not query code
- Special characters are escaped automatically
- No SQL injection possible

---

## Summary Table

| Attack | Type | Impact | Protection |
|--------|------|--------|-----------|
| CSRF | Session-based | Unwanted requests | CSRF Token |
| XSS | Client-side | Cookie theft, defacement | Input escaping, validation |
| CORS | Origin-based | Unauthorized requests | Origin whitelisting |
| SQL Injection | Database | Data breach, deletion | Parameterized queries |