# OWASP Top 10 Vulnerabilities - Detailed Analysis

## 1. SQL Injection

### What is it?
SQL Injection is when an attacker injects SQL code into user input fields to manipulate the database.

### Severity
**CRITICAL** - OWASP Rank #3

### How I Exploited It
On TryHackMe, I entered: `admin' OR '1'='1'--` in the login form
- Result: Logged in as admin without knowing the password
- Why it worked: User input was directly concatenated into SQL query

### Real-World Impact
- Complete database breach
- Unauthorized access to user accounts
- Customer data stolen
- Regulatory fines

### How to Fix It
```python
#  VULNERABLE
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"

# SECURE - Use parameterized queries
query = "SELECT * FROM users WHERE username = ? AND password = ?"
result = db.execute(query, (username, password))
```

**Why this works:** User input is separated from SQL code, preventing injection.

---

## 2. Cross-Site Scripting (XSS)

### What is it?
XSS is when attacker injects malicious JavaScript code that executes in victim's browser.

### Severity
**HIGH** - OWASP Rank #4

### How I Exploited It
On TryHackMe, I injected: `<script>alert('XSS')</script>`
- Result: JavaScript executed in browser
- Impact: Could steal cookies, redirect users, steal credentials

### Real-World Impact
- Session hijacking
- Credential theft
- Malware distribution
- Website defacement

### How to Fix It
```python
from flask import escape

# VULNERABLE
return f"<h1>Welcome, {user_input}</h1>"

# SECURE - Escape HTML
return f"<h1>Welcome, {escape(user_input)}</h1>"
```

**Why this works:** Special characters are converted to HTML entities, preventing script execution.

---

## 3. Cross-Site Request Forgery (CSRF)

### What is it?
CSRF tricks logged-in users into making unwanted requests without their knowledge.

### Severity
**MEDIUM** - OWASP Rank #7

### How I Exploited It
Created a hidden form that transferred money when user clicked it.
- User didn't realize what was happening
- Request was sent with their authentication cookies
- Action was performed without consent

### Real-World Impact
- Money transfers without permission
- Account password changes
- Email address changes
- Account locked out

### How to Fix It
```html
<!--  SECURE - Add CSRF token to form -->
<form action="/transfer" method="POST">
    <input type="hidden" name="csrf_token" value="unique_token_12345">
    <input type="number" name="amount">
    <button>Transfer</button>
</form>
```

Server verifies token before processing:
```python
@app.route('/transfer', methods=['POST'])
def transfer():
    token = request.form.get('csrf_token')
    if not verify_token(token):
        return "CSRF Protection: Request rejected", 403
    # Process transfer
```

**Why this works:** Attacker can't guess the unique token, so their requests are rejected.

---

## 4. Broken Authentication

### What is it?
Weak or missing authentication controls that allow attackers to compromise user accounts.

### Severity
 **HIGH** - OWASP Rank #2

### How I Exploited It
- No rate limiting: Tried thousands of passwords without being blocked
- Weak passwords: Simple passwords like "admin/admin" worked
- No password hashing: Passwords stored in plaintext

### Real-World Impact
- Account takeover
- Identity theft
- Access to sensitive data
- Financial fraud

### How to Fix It
```python
from werkzeug.security import generate_password_hash, check_password_hash

# SECURE - Hash passwords
hashed_password = generate_password_hash(password, method='pbkdf2:sha256')
user = User(username=username, password=hashed_password)

# Verify password
if check_password_hash(user.password, provided_password):
    # Login successful
```

Add rate limiting:
```python
from flask_limiter import Limiter

@app.route('/login', methods=['POST'])
@limiter.limit("5 per minute")  # Max 5 attempts per minute
def login():
    # Login logic
```

**Why this works:** 
- Passwords are hashed, not plaintext
- Rate limiting prevents brute force attacks
- Even if database is stolen, passwords are unusable

---

## 5. Insecure Direct Object References (IDOR)

### What is it?
Application exposes direct references to objects without checking permissions.

### Severity
**MEDIUM** - OWASP Rank #5

### How I Exploited It
Changed URL from `/profile/5` to `/profile/1`, `/profile/2`, etc.
- Could see other users' private profiles
- Accessed sensitive personal information
- No permission checks

### Real-World Impact
- Exposure of personal information (PII)
- Financial fraud
- Identity theft
- Regulatory violations (GDPR)

### How to Fix It
```python
# VULNERABLE - No permission check
@app.route('/profile/<user_id>')
def view_profile(user_id):
    user = User.query.get(user_id)
    return render_template('profile.html', user=user)

# SECURE - Check permission first
@app.route('/profile/<user_id>')
def view_profile(user_id):
    current_user_id = session.get('user_id')
    
    # Only allow viewing own profile
    if current_user_id != user_id:
        return "Access Denied", 403
    
    user = User.query.get(user_id)
    return render_template('profile.html', user=user)
```

**Why this works:** Server checks permission before returning any data.

---

## Summary

| Vulnerability | Fix Strategy |
|---|---|
| SQL Injection | Use parameterized queries |
| XSS | Escape HTML output |
| CSRF | Add CSRF tokens |
| Broken Auth | Hash passwords + rate limiting |
| IDOR | Check permissions before showing data |

## Key Principle
**Never trust user input. Always validate, sanitize, and check permissions.**

## Next Steps
- [ ] Add screenshots from TryHackMe
- [ ] Deploy on cloud
- [ ] Get certificates
- [ ] Interview preparation

**Project Status: 50% Complete** 
