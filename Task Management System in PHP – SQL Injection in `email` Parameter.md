# Task Management System in PHP – SQL Injection in `email` Parameter

## Credit

**Discovered by:** Ahmad Marzook

## Product

Task Management System in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/task-management-system-in-php-with-source-code/

## Affected Version

Version not specified.

The exact affected version should be verified from the tested application package before formal CVE or VulDB submission.

## Vulnerability Type

SQL Injection / Boolean-Based SQL Injection

## CWE

**CWE-89 – Improper Neutralization of Special Elements Used in an SQL Command ('SQL Injection')**

## OWASP Top 10

**A03:2021 – Injection**

## Severity

**High**

## Suggested CVSS v3.1 Score

**8.1 (High)**

```text id="s9twoc"
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
```

The final CVSS score should be adjusted according to confirmed exploitation results. If the vulnerability only affects authentication logic without providing broader database access, a different impact assessment may be appropriate.

---

# Vulnerability Overview

The Task Management System in PHP contains a potential SQL injection vulnerability in the authentication functionality available through the application's `index.php` endpoint.

The vulnerability affects the following POST parameter:

```text id="wqv9ai"
email
```

The affected endpoint is:

```text id="g8l5kb"
POST /PROJECT_NEW/index.php
```

The application accepts an email address and password as authentication credentials. Testing indicates that SQL syntax supplied through the `email` parameter may be interpreted as part of the backend authentication query instead of being handled exclusively as user data.

The supplied test value contains a boolean SQL expression:

```text id="8pnpdu"
anas@gmail.com93122137' or '7333'='7333
```

The expression:

```text id="asjohf"
'7333'='7333
```

evaluates to true.

If the application directly concatenates the supplied `email` value into an SQL authentication query, the injected `OR` condition may modify the logical behavior of the resulting SQL statement.

Depending on the exact structure of the authentication query and placement of parentheses, successful exploitation could potentially affect login validation or demonstrate that arbitrary SQL conditions can be introduced into the backend query.

---

# Description

The Task Management System in PHP is affected by a potential SQL injection vulnerability in the application's login functionality.

The issue exists in the `email` parameter processed by:

```text id="2v8id6"
/PROJECT_NEW/index.php
```

During authentication, the application receives an email address and password through an HTTP POST request. These values are subsequently used to determine whether the supplied credentials correspond to a valid application account.

The `email` parameter appears to be incorporated into a backend SQL query without sufficient use of prepared statements or parameter binding. As a result, an attacker may be able to introduce SQL syntax into the value and influence the logical structure of the authentication query.

The provided proof-of-concept request supplies the following value:

```text id="kr67k2"
anas@gmail.com93122137' or '7333'='7333
```

This input introduces a boolean condition that always evaluates to true:

```text id="y7ez5b"
'7333'='7333
```

If the application constructs its SQL authentication statement through direct string concatenation, the injected quotation mark can terminate the intended email string and the subsequent SQL expression can become part of the database query.

A vulnerable authentication pattern may conceptually resemble:

```php id="kl62dt"
$email = $_POST['email'];
$password = $_POST['password'];

$sql = "SELECT * FROM users
        WHERE email = '$email'
        AND password = '$password'";

$result = mysqli_query($conn, $sql);
```

When attacker-controlled input is inserted directly into such a query, SQL operators included in the input may change the intended query logic.

Depending on the precise SQL statement, operator precedence, password condition, and comment handling, successful exploitation could potentially result in authentication bypass. Even where the tested boolean expression does not bypass authentication directly, the ability to influence SQL query logic may indicate a broader SQL injection vulnerability.

If arbitrary SQL injection is confirmed, an unauthenticated attacker may potentially use the vulnerability to retrieve sensitive information from the database, enumerate application accounts, access authentication-related data, or manipulate database records where the database account has sufficient privileges.

Because the affected functionality is located on the application's public login interface, an attacker may be able to interact with the vulnerable parameter without possessing an existing authenticated account.

The root cause is improper separation of untrusted user input from SQL commands. Authentication queries should use prepared statements with parameter binding, and passwords should be securely hashed and verified independently rather than included directly in dynamically constructed SQL queries.

---

# Root Cause

The vulnerability is caused by unsafe handling of user-controlled authentication input.

Potential contributing factors include:

* Direct concatenation of the `email` parameter into an SQL query.
* Failure to use prepared statements.
* Failure to use parameter binding.
* Insufficient server-side validation.
* Insecure construction of authentication queries.
* Potentially unsafe handling of password verification.

For example, the following pattern would be vulnerable:

```php id="tx1scx"
$email = $_POST['email'];
$password = $_POST['password'];

$query = "SELECT * FROM users
          WHERE email='$email'
          AND password='$password'";

$result = mysqli_query($conn, $query);
```

The correct implementation should ensure that the supplied email address is always treated as data rather than executable SQL syntax.

The exact vulnerable source-code statement should be included in the final advisory if source-code analysis is available.

---

# Affected Endpoint

```text id="3ihjvw"
POST /PROJECT_NEW/index.php
```

## Vulnerable Parameter

```text id="kt1a60"
email
```

## Additional Parameters

```text id="0a7lrg"
password
submit
```

## HTTP Method

```text id="dg5t83"
POST
```

---

# Proof of Concept

## Full HTTP Request

```http id="p4gv6t"
POST /PROJECT_NEW/index.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="146", "Not=A?Brand";v="8", "Chromium";v="146"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=nq4f1fni8pdjjf3jf8e779psji
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/PROJECT_NEW/index.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 43

email=anas@gmail.com93122137'%20or%20'7333'%3d'7333&password=123&submit=
```

Decoded request body:

```text id="5c2gfh"
email=anas@gmail.com93122137' or '7333'='7333
password=123
submit=
```

The injected boolean condition is:

```text id="3p8a99"
'7333'='7333
```

This condition evaluates to true.

---

# Steps to Reproduce

1. Deploy the Task Management System in PHP in an authorized local testing environment.

2. Navigate to:

```text id="mb7v7g"
/PROJECT_NEW/index.php
```

3. Intercept the login request using an HTTP testing proxy.

4. Establish a baseline by attempting to authenticate using invalid credentials and record the application's response.

5. Modify the `email` parameter to include the tested boolean SQL expression.

6. Submit the modified request.

7. Compare the modified response against the invalid-login baseline.

8. Examine differences including:

   * HTTP status code
   * Redirect location
   * Response length
   * Authentication state
   * Session state
   * Application content

9. If the injected request produces a query-dependent response different from the baseline, review the corresponding authentication source code to determine whether the `email` parameter is directly incorporated into an SQL query.

10. If the request results in successful authentication without valid credentials, document the issue as SQL injection leading to authentication bypass.

---

# Expected Result

The application should treat the complete contents of the `email` parameter exclusively as an email address.

SQL operators and quotation characters included in the value should not alter the structure or logical behavior of the backend database query.

Invalid credentials should consistently result in authentication failure.

---

# Observed Result

The supplied request introduces an SQL boolean expression into the `email` parameter.

If this input causes a reproducible difference in authentication behavior compared with an equivalent invalid-login request, it indicates that the user-controlled parameter may influence the backend SQL query.

If the request successfully authenticates without valid credentials, the vulnerability additionally results in an authentication bypass.

The exact observed result should be documented before formal vulnerability submission.

---

# Impact

If SQL injection is confirmed, successful exploitation may allow an unauthenticated attacker to:

* Manipulate backend authentication queries.
* Potentially bypass authentication where query structure permits.
* Gain unauthorized access to application functionality.
* Enumerate database information.
* Retrieve user or administrator account information.
* Access authentication-related data.
* Read sensitive task and project information.
* Modify application data where database permissions permit.
* Delete database records where database permissions permit.

If authentication bypass is confirmed and the application returns a privileged account, the attacker may gain access to functionality intended only for authorized users.

The practical impact depends on the vulnerable query, database permissions, application authorization controls, and the privileges associated with any account obtained through authentication bypass.

---

# Attack Requirements

Assuming the login page is publicly accessible:

```text id="68gqvb"
Attack Vector: Network
Privileges Required: None
User Interaction: None
```

No existing authenticated account is required to submit input to the affected login endpoint.

---

# Vulnerability Classification

**CWE**

CWE-89 – Improper Neutralization of Special Elements Used in an SQL Command ('SQL Injection')

**OWASP Top 10**

A03:2021 – Injection

---

# Remediation

## Use Prepared Statements

Authentication queries must use prepared statements and parameter binding.

Example using MySQLi:

```php id="qmx2bu"
$email = $_POST['email'] ?? '';
$password = $_POST['password'] ?? '';

if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    exit('Invalid credentials.');
}

$stmt = $conn->prepare(
    'SELECT id, email, password_hash
     FROM users
     WHERE email = ?
     LIMIT 1'
);

$stmt->bind_param('s', $email);
$stmt->execute();

$result = $stmt->get_result();
$user = $result->fetch_assoc();

if ($user && password_verify($password, $user['password_hash'])) {
    // Authentication successful.
} else {
    // Authentication failed.
}
```

## Secure Password Handling

Passwords should be stored using PHP's secure password-hashing functionality:

```text id="6gdz77"
password_hash()
```

Passwords should be verified using:

```text id="dq2fz4"
password_verify()
```

Passwords should not be stored in plaintext or directly compared as part of dynamically constructed SQL queries.

## Input Validation

Validate email addresses using server-side validation.

For example:

```php id="lcmw54"
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    exit('Invalid email address.');
}
```

Input validation provides defense in depth but must not replace prepared statements.

## Apply Least-Privilege Database Permissions

The application's database account should receive only the permissions required for normal operation.

## Disable Detailed Database Errors

Database errors should not be returned directly to unauthenticated users.

## Review All SQL Queries

Developers should review the entire application for similar SQL injection vulnerabilities, particularly in:

* Authentication endpoints
* Registration functionality
* Task management
* Project management
* User administration
* Search functionality
* Record modification and deletion endpoints

All user-controlled values should be handled using parameterized queries.

---

# Conclusion

The Task Management System in PHP contains a potential SQL injection vulnerability in the `email` parameter processed by the `/PROJECT_NEW/index.php` login endpoint.

The supplied proof-of-concept introduces an always-true boolean SQL expression into the email value. If the application incorporates this parameter directly into an SQL authentication query, attacker-controlled SQL syntax may alter the intended query logic.

If testing confirms that the injected condition produces a reproducible query-dependent response, the vulnerability should be classified as SQL injection under CWE-89. If the request additionally permits login without valid credentials, the impact should explicitly document authentication bypass.

The vulnerability should be remediated by implementing prepared statements and parameter binding, validating authentication input, securely hashing passwords, and reviewing all database interactions for similar unsafe query construction.


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/3065258c-ac98-4d28-8d9a-13249b0fe89c" />
