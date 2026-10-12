Web Application Security: Learning Notes
Name: Abbas Khan  |
Reg No: 25849  |  
Module 3 (Day 11 to Day 15)
Note: the lab entries in Part D are sample values for a typical DVWA or OWASP Juice Shop lab. Replace them with your own results and screenshots before you submit.
Part A: How Web Applications Work
1. Web Application Architecture
A web application has three main tiers:
Tier
Role
Examples
Client (front end)
What the user sees and interacts with
Browser, HTML, CSS, JavaScript
Server (back end)
Business logic and request handling
Apache, Nginx, PHP, Node.js, Python, Java
Database
Stores persistent data
MySQL, PostgreSQL, MongoDB
Key idea for security: the browser is controlled by the user, so anything the client sends (forms, headers, cookies, hidden fields) can be changed and must never be trusted. Validation and access checks must happen on the server.
2. HTTP Request / Response Lifecycle
1. The user enters a URL; DNS resolves the domain to an IP address.
2. The browser opens a TCP connection (and a TLS handshake for HTTPS).
3. The browser sends an HTTP request.
4. The server processes it (may query the database) and sends an HTTP response.
5. The browser renders the page and may send more requests for images, scripts and APIs.
2.1 Request Structure
POST /login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
Content-Type: application/x-www-form-urlencoded

username=alice&password=secret
• Request line: method, path, HTTP version.
• Headers: metadata (Host, Cookie, User-Agent, Content-Type, Authorization).
• Body: data sent with POST/PUT requests.
2.2 Response Structure
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=abc123; HttpOnly; Secure

<html>...</html>
2.3 Methods and Status Codes
Method
Purpose
GET
Retrieve data (parameters appear in the URL)
POST
Submit data in the body
PUT / PATCH
Update data
DELETE
Remove data
Code
Meaning
200
OK
301 / 302
Redirect
400
Bad request
401
Unauthorized (not authenticated)
403
Forbidden (not allowed)
404
Not found
500
Server error (may leak details if not handled)
2.4 Important Security Headers
Header
Protects against
Content-Security-Policy
XSS and unwanted content loading
X-Frame-Options / frame-ancestors
Clickjacking
Strict-Transport-Security
Downgrade to HTTP (forces HTTPS)
X-Content-Type-Options: nosniff
MIME type sniffing
Referrer-Policy
Leaking URLs to other sites
3. Cookies and Sessions
HTTP is stateless, so websites use cookies and sessions to remember users.
• Cookie: small data stored in the browser and sent with each request to that site.
• Session: server-side record of a logged-in user, identified by a session ID stored in a cookie.
Cookie security flags:
Flag
Effect
HttpOnly
JavaScript cannot read the cookie (limits XSS theft)
Secure
Sent only over HTTPS
SameSite (Strict / Lax)
Limits cross-site sending (helps against CSRF)
Expires / Max-Age
Limits how long the cookie lives
Session attacks: session hijacking (stealing the ID), session fixation (forcing a known ID), and weak or predictable IDs.
Defenses: generate long random session IDs, regenerate the ID after login, expire sessions on logout and timeout, and use HTTPS.
4. Authentication vs Authorization
Concept
Question
Examples
Authentication
Who are you?
Password, MFA, biometrics, OAuth login
Authorization
What may you do?
Roles, permissions, access control lists
• Common authentication methods: passwords, tokens (JWT), API keys, OAuth 2.0 / OpenID Connect, MFA.
• Access control models: RBAC (role-based), ABAC (attribute-based), least privilege.
• Rule: always check authorization on the server for every request, not only in the user interface.
Part B: OWASP Top 10 Analysis
The OWASP Top 10 lists the most critical web application security risks. The table below follows the 2025 edition; confirm the current list and wording at owasp.org/Top10 before submitting.
5. Overview Table
#
Category
What it means
Example attack
Main defense
A01
Broken Access Control
Users can act outside their permissions
Changing ?id=5 to ?id=6 to view another user's data (IDOR)
Server-side checks, deny by default, least privilege
A02
Security Misconfiguration
Insecure defaults, extra features, verbose errors
Default admin password, directory listing, debug mode on
Hardening, remove defaults, automated config checks
A03
Software Supply Chain Failures
Vulnerable or malicious dependencies and build processes
Compromised third-party package
Track dependencies, scan, pin versions, verify sources
A04
Cryptographic Failures
Weak or missing protection of sensitive data
Passwords stored in plain text, HTTP instead of HTTPS
Strong algorithms, TLS everywhere, salted password hashing
A05
Injection
Untrusted input is run as a command or query
' OR '1'='1 in a login form (SQL injection)
Parameterized queries, input validation, output encoding
A06
Insecure Design
Flaws in the design itself, not just the code
No rate limit on password reset
Threat modeling, secure design patterns
A07
Authentication Failures
Weak login, session or credential handling
Brute force, credential stuffing, predictable session IDs
MFA, strong passwords, rate limiting, secure sessions
A08
Software or Data Integrity Failures
Code or data trusted without verification
Insecure deserialization, unsigned updates
Digital signatures, integrity checks, safe deserialization
A09
Security Logging and Alerting Failures
Attacks go unnoticed
No logs for failed logins
Log key events, monitor and alert, protect logs
A10
Mishandling of Exceptional Conditions
Errors and unexpected states handled badly
Stack trace shown to users, app fails open
Handle errors safely, fail closed, generic error pages
6. Deeper Look at the Most Tested Vulnerabilities
6.1 SQL Injection (Injection)
Vulnerable code idea:
query = "SELECT * FROM users WHERE name='" + user_input + "'"
Attack input: ' OR '1'='1 changes the query so it returns every user.
Fix: parameterized queries (prepared statements):
cursor.execute("SELECT * FROM users WHERE name = %s", (user_input,))
Also use least-privilege database accounts and never show raw database errors.
6.2 Cross-Site Scripting, XSS (Injection)
Type
How it works
Reflected
Malicious script in a URL parameter is returned in the page
Stored
Script is saved (e.g., in a comment) and runs for every visitor
DOM-based
Client-side JavaScript writes untrusted data into the page
Test input (lab only): <script>alert(1)</script>
Fix: encode output for the right context, validate input, use a Content-Security-Policy, set HttpOnly cookies.
6.3 Broken Access Control
• IDOR: changing an object ID in the URL or request to reach another user's data.
• Privilege escalation: a normal user reaches admin functions (e.g., /admin).
• Forced browsing: guessing hidden URLs.
Fix: check permissions on the server for every request, use indirect or unguessable references, deny by default.
6.4 Authentication Failures
• Weak or default passwords, no lockout, no MFA.
• Session IDs in URLs or never expiring.
• Credentials sent over HTTP.
Fix: MFA, rate limiting and lockout, strong password policy, secure session handling, HTTPS.
6.5 CSRF (Cross-Site Request Forgery)
A malicious site makes the victim's browser send an authenticated request to another site. Fix: anti-CSRF tokens and SameSite cookies.
6.6 Other Common Issues
• Command injection: user input reaches a system shell command. Fix: avoid shell calls, use safe APIs, allowlist input.
• File upload flaws: executable files uploaded. Fix: check type and content, store outside the web root, rename files.
• SSRF: the server is tricked into requesting internal URLs. Fix: allowlist destinations, block internal address ranges.
Part C: Secure Coding Practices
1. Validate all input on the server: type, length, format, range. Prefer allowlists over blocklists.
2. Encode output for its context (HTML, JavaScript, URL, SQL).
3. Use parameterized queries and safe framework features, never string-built queries.
4. Enforce access control on the server for every request; deny by default.
5. Handle authentication properly: hash passwords with bcrypt, scrypt or Argon2 with salt; add MFA and rate limiting.
6. Protect sessions: random IDs, regenerate after login, HttpOnly, Secure, SameSite, timeouts.
7. Use HTTPS everywhere with the Strict-Transport-Security header.
8. Handle errors safely: generic messages for users, details only in protected logs.
9. Keep dependencies updated and scan them for known vulnerabilities.
10. Never hard-code secrets (keys, passwords); use environment variables or a secrets manager.
11. Log and monitor security events (logins, failures, access denials).
12. Apply least privilege to users, services and database accounts.
Part D: Secure Web Application Lab (Identify, Test, Fix, Document)
Safety rule: test only applications you own or have permission to test. Use a local sandbox such as DVWA, OWASP Juice Shop or WebGoat on an isolated network, never real websites.
7. Lab Setup
Item
Detail
Target application
DVWA (Damn Vulnerable Web Application) running in a VM or Docker container
Security level
Set to Low for testing, then compare with Medium and High
Attacker tools
Browser, Burp Suite, OWASP ZAP
Network
Host-Only or local network only, no internet exposure
Snapshot taken
Yes, before testing
Screenshot: (insert DVWA login page and security level setting)
8. The Four-Step Method
1. Identify: map the application (pages, forms, parameters, cookies, user roles) and list possible weak points.
2. Test: try safe, controlled attacks in the sandbox (Burp Repeater, ZAP scans, manual inputs).
3. Fix: patch the code or configuration for each confirmed issue.
4. Document: record evidence, impact, severity and the fix in a report.
9. Vulnerability Report
ID
Vulnerability
OWASP category
Where found
Test performed
Severity
V1
SQL Injection
A05 Injection
DVWA, SQL Injection page, id parameter
Entered 1' OR '1'='1 and all user records were returned
High
V2
Reflected XSS
A05 Injection
DVWA, XSS (Reflected) page, name field
Entered <script>alert(1)</script> and the alert ran
High
V3
Insecure Direct Object Reference
A01 Broken Access Control
User profile URL with an ID
Changed the ID in the request to view another user's data
High
V4
Weak authentication controls
A07 Authentication Failures
Login form
No lockout after repeated wrong passwords
Medium
V5
Missing security headers
A02 Security Misconfiguration
All pages
ZAP reported no CSP and no X-Frame-Options
Medium
V6
Cookie without HttpOnly
A07 Authentication Failures
Session cookie
Cookie readable with document.cookie
Low
Screenshot evidence: (insert Burp request, ZAP alerts, and XSS alert box)
10. Detailed Finding Example (V1: SQL Injection)
Field
Details
Description
User input is placed directly into a database query without sanitizing
Steps to reproduce
Open the SQL Injection page, enter 1' OR '1'='1, submit
Impact
Attacker can read all user records and may modify or delete data
Evidence
Screenshot of the full user list returned
Fix
Use prepared statements, validate that the input is numeric, limit database permissions
Retest result
After the fix, the same input returns no extra records
11. Remediation Recommendations
ID
Fix
Priority
V1
Replace string-built SQL with parameterized queries; use a least-privilege database account
High
V2
Encode all output; add a Content-Security-Policy; validate input
High
V3
Check ownership of the requested object on the server for every request
High
V4
Add rate limiting, account lockout and MFA; enforce a strong password policy
Medium
V5
Add Content-Security-Policy, X-Frame-Options, Strict-Transport-Security, X-Content-Type-Options headers
Medium
V6
Set HttpOnly, Secure and SameSite on session cookies
Low
Before and after example (output encoding):
Before: echo "Hello " . $_GET['name'];
After:  echo "Hello " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
12. Demo Walkthrough Plan
Record a 3 to 5 minute screen demo (Zoom, OBS or a phone screen recorder) and upload it to the GitHub repo or a private YouTube link.
1. Intro (30 sec): state the goal and show the isolated lab.
2. Identify (45 sec): map the app in Burp or ZAP.
3. Test (90 sec): show SQL injection, XSS and one access control issue.
4. Fix (60 sec): show the code change and the secure version.
5. Retest (30 sec): show the attack no longer works.
6. Wrap-up (15 sec): summarize the findings and what you learned.
13. Suggested GitHub Repository Structure
web-app-security-lab/
├── README.md
├── 01-owasp-analysis/
│   └── owasp-top10-analysis.md
├── 02-lab-setup/
│   └── setup-notes.md
├── 03-vulnerability-report/
│   └── vulnerability-report.md
├── 04-remediation/
│   ├── recommendations.md
│   └── code-fixes/
├── 05-demo/
│   └── walkthrough-link.md
└── screenshots/
Never commit passwords, API keys or real personal data.
Key Takeaways
• Never trust client-side data: validate and authorize on the server.
• Most serious web flaws (injection, broken access control, weak authentication) come from missing server-side checks.
• Secure cookies and sessions protect logged-in users.
• Finding a bug is only half the job: fixing, retesting and documenting complete it.
• Test only in a legal, isolated sandbox.
Challenges & Reflection
• Challenges I faced: Setting up DVWA and configuring the browser to work with the Burp proxy.
• How I solved them: Checked the database setup page in DVWA and installed the Burp certificate in the browser.
• What I want to learn next: API security testing and secure code review.
