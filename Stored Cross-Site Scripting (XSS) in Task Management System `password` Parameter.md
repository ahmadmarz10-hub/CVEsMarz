# Stored Cross-Site Scripting (XSS) in Task Management System `password` Parameter

## Credit

**Discovered by:** Ahmad Marzouk

---

## Product

Task Management System in PHP

---

## Vendor

Code-Projects

---

## Vendor URL

https://code-projects.org/task-management-system-in-php-with-source-code/

---

## Affected Version

Version not specified.

The exact affected version should be confirmed from the tested application package before formal CVE or VulDB submission.

---

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

---

## CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

---

## OWASP Top 10

**A03:2021 – Injection**

---

## Severity

**Medium**

---

## Suggested CVSS v3.1 Score

**5.4 (Medium)**

```text
CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N
```

The final CVSS score depends on where the stored password value is later displayed and which users can access that page.

---

# Vulnerability Overview

The Task Management System in PHP contains a potential Stored Cross-Site Scripting vulnerability in the user password update functionality.

The affected endpoint is:

```text
POST /PROJECT_NEW/user/UpdateUserProfile.php
```

The vulnerability affects the:

```text
password
```

parameter.

An authenticated user can submit a value containing HTML or JavaScript markup through the password update functionality.

The supplied request contains the following JavaScript payload:

```html
<script>alert(1)</script>
```

embedded within the `password` parameter.

If the application stores the submitted password as plaintext or otherwise preserves the original input and subsequently renders that value in an HTML page without contextual output encoding, the injected JavaScript may execute in the browser of a user viewing the affected record.

Because this finding depends on the password being stored and later displayed, successful persistence and execution must be confirmed before classifying the issue as Stored XSS.

---

# Description

The Task Management System in PHP is potentially vulnerable to Stored Cross-Site Scripting in the password update functionality implemented by the `UpdateUserProfile.php` endpoint.

The application allows an authenticated user to modify their account password through the `password` POST parameter. The supplied proof-of-concept request places JavaScript markup inside the submitted password value.

The decoded value is:

```text
123dbjw0<script>alert(1)</script>zdah4
```

The embedded JavaScript payload is:

```html
<script>alert(1)</script>
```

If the application stores the original password value directly in the database without secure password hashing and later displays that value in an administrative interface, profile-management page, user listing, debugging interface, or another HTML response, the browser may interpret the injected `<script>` element as active JavaScript.

In such a scenario, an authenticated attacker could update their password with a malicious payload and cause the application to persist the injected code. When another user, such as an administrator, subsequently views a page containing the stored password value, the JavaScript would execute within the security context of the application.

Successful exploitation could allow execution of arbitrary client-side JavaScript in the browser of affected users. Depending on the victim's privileges and application security controls, this could enable unauthorized actions using the victim's active session, manipulation of displayed content, access to data available to client-side scripts, or phishing attacks.

However, passwords should never normally be rendered back to users in plaintext. A securely designed application should hash passwords using a one-way password hashing algorithm and never retain or display the original password value.

Therefore, this vulnerability should only be classified as Stored XSS if testing confirms all of the following conditions:

1. The malicious password value is accepted.
2. The original value is stored persistently.
3. The stored value is later rendered into an HTML response.
4. The JavaScript payload executes in a browser.

If the application stores the password using a secure one-way hash such as PHP's `password_hash()` and never displays the original password, the submitted JavaScript payload cannot result in Stored XSS through this parameter.

---

# Root Cause

If confirmed, the vulnerability would result from insecure password handling combined with missing contextual output encoding.

Potential contributing factors include:

* Storing passwords in plaintext.
* Preserving attacker-controlled password input without secure hashing.
* Displaying stored password values in the application interface.
* Rendering stored values without HTML output encoding.
* Insufficient validation of sensitive account fields.

A vulnerable implementation could resemble:

```php
$password = $_POST['password'];

$sql = "UPDATE users
        SET password='$password'
        WHERE id='$user_id'";

mysqli_query($conn, $sql);
```

Followed by unsafe output elsewhere in the application:

```php
echo $row['password'];
```

This design presents two separate security problems:

1. Insecure plaintext password storage.
2. Stored XSS if the value is rendered without encoding.

The exact source code should be reviewed before publication.

---

# Affected Endpoint

```text
POST /PROJECT_NEW/user/UpdateUserProfile.php
```

---

# Vulnerable Parameter

```text
password
```

---

# Supporting Parameter

```text
updatePassword
```

---

# Authentication Requirements

The supplied request contains an active PHP session cookie, indicating that the password update functionality likely requires authentication.

Likely attack requirements:

```text
Privileges Required: Low
User Interaction: Required
```

The attacker must be able to modify their password, and another user must later view a page where the original stored value is rendered.

---

# Proof of Concept

## Full HTTP Request

```http
POST /PROJECT_NEW/user/UpdateUserProfile.php HTTP/1.1
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
Cookie: PHPSESSID=o5ffcsk8u7ira4lbe8bvtbgbbi
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/PROJECT_NEW/user/UpdateUserProfile.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 28

password=123dbjw0%3cscript%3ealert(1)%3c%2fscript%3ezdah4&updatePassword=
```

Decoded request body:

```text
password=123dbjw0<script>alert(1)</script>zdah4
updatePassword=
```

The embedded test payload is:

```html
<script>alert(1)</script>
```

---

# Steps to Reproduce

1. Deploy the Task Management System in PHP in an authorized local testing environment.

2. Authenticate with a valid test account.

3. Navigate to:

```text
/PROJECT_NEW/user/UpdateUserProfile.php
```

4. Intercept the password update request.

5. Submit a password value containing the test JavaScript payload.

6. Confirm whether the application accepts the password change.

7. Inspect the database or relevant application behavior to determine whether the original value is stored or converted into a password hash.

8. Navigate to any authorized page where the password value is displayed.

9. Observe whether the stored JavaScript executes.

10. Repeat the page load to confirm that the value is persistent.

If the application stores only a password hash and never renders the original password value, the Stored XSS condition is not met.

---

# Expected Result

The application should securely hash the submitted password before storage.

The original password value should never be:

* Stored in plaintext.
* Returned in an HTTP response.
* Displayed in an administrative interface.
* Rendered in HTML.

A submitted password containing HTML markup should be treated exclusively as password data and converted into a secure password hash before storage.

---

# Observed Result

The supplied request demonstrates that the `password` parameter accepts a value containing JavaScript markup.

Stored XSS is confirmed only if the original malicious input is persisted and subsequently rendered without output encoding.

If the application stores the value in plaintext but does not render it, this may instead indicate an **insecure password storage** vulnerability rather than XSS.

---

# Impact

If Stored XSS is confirmed, successful exploitation may allow an authenticated attacker to execute arbitrary JavaScript in the browser of users who view the stored value.

Potential consequences include:

* Unauthorized actions using the victim's active session.
* Manipulation of application content.
* Access to information exposed to client-side scripts.
* Phishing or deceptive interface injection.
* Targeting privileged administrators.

Additionally, if the application stores passwords in plaintext, compromise of the database could expose user credentials directly and create a separate critical security risk.

---

# Vulnerability Classification

## CWE

**CWE-79 – Cross-Site Scripting**

If plaintext password storage is also confirmed, the application may additionally be affected by:

**CWE-256 – Plaintext Storage of a Password**

or another applicable insecure credential-storage classification.

---

# Remediation

## Securely Hash Passwords

Passwords must be stored using a secure one-way password hashing function.

PHP example:

```php
$password = $_POST['password'] ?? '';

if (strlen($password) < 8) {
    exit('Password does not meet security requirements.');
}

$passwordHash = password_hash(
    $password,
    PASSWORD_DEFAULT
);

$stmt = $conn->prepare(
    'UPDATE users SET password = ? WHERE id = ?'
);

$stmt->bind_param(
    'si',
    $passwordHash,
    $userId
);

$stmt->execute();
```

Authentication should use:

```php
password_verify($submittedPassword, $storedHash);
```

---

## Never Display Passwords

The application should never display:

* Current passwords.
* Previous passwords.
* Plaintext password values.

Password fields in account settings should remain empty rather than being populated with stored values.

---

## Apply Output Encoding

Any user-controlled data that is displayed in HTML should be contextually encoded.

For example:

```php
echo htmlspecialchars(
    $value,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

This is defense against XSS but is not a substitute for proper password hashing.

---

## Use Prepared Statements

All password-update database operations should use parameterized SQL statements.

This protects the update functionality against SQL injection.

---

## Review Existing Password Storage

If the application currently stores plaintext passwords:

* Migrate users to secure password hashes.
* Force password resets where appropriate.
* Remove plaintext credentials from backups and logs.
* Review whether plaintext passwords were previously exposed.

---

# Conclusion

The Task Management System in PHP accepts JavaScript markup through the `password` parameter of `/PROJECT_NEW/user/UpdateUserProfile.php`.

This condition represents Stored Cross-Site Scripting only if the original password value is persisted and later rendered into an HTML response without contextual output encoding.

Because securely designed applications hash passwords and never display their original values, confirmation of persistence and execution is essential before submitting this issue as CWE-79.

If the application instead stores the submitted password directly in plaintext, that behavior should also be documented separately as an insecure password-storage vulnerability. Developers should implement `password_hash()` and `password_verify()`, use prepared statements, and ensure that password values are never rendered back to users.



<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/f98f924a-f382-498f-a93e-8ff8e6f528e1" />
