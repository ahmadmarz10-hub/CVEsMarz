# Stored Cross-Site Scripting (XSS) in Task Management System `lname` Parameter

## Credit

**Discovered by:** Ahmad Marzook

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

The final CVSS score should be adjusted depending on which users can view the affected profile and whether privileged administrators are exposed to the stored payload.

---

# Vulnerability Overview

The Task Management System in PHP contains a potential Stored Cross-Site Scripting vulnerability in the user profile update functionality.

The affected endpoint is:

```text
POST /PROJECT_NEW/user/UpdateUserProfile.php
```

The vulnerability affects the:

```text
lname
```

parameter.

An authenticated user can submit HTML or JavaScript content through the last-name field. The supplied proof-of-concept request contains the following payload:

```html
<script>alert(1)</script>
```

embedded within the `lname` value.

If the application stores the submitted last name in the backend database and subsequently renders the value in a profile page, task-management interface, administrative user list, or another HTML response without applying appropriate contextual output encoding, the injected JavaScript is executed by the browser of any user who views the affected record.

Because the payload persists after the initial request and may execute during subsequent page views, the vulnerability is classified as a Stored or Persistent Cross-Site Scripting vulnerability.

---

# Description

The Task Management System in PHP is vulnerable to Stored Cross-Site Scripting in the user profile update functionality implemented by the `UpdateUserProfile.php` endpoint.

The application allows authenticated users to modify profile information through the `fname` and `lname` parameters. The `lname` parameter accepts user-controlled input that may contain HTML and JavaScript markup.

The supplied request submits a last-name value containing:

```html
<script>alert(1)</script>
```

If this value is accepted and stored by the application without adequate validation or sanitization, it becomes persistent application data.

When the stored last name is subsequently retrieved from the database and inserted into an HTML page without proper contextual output encoding, the browser interprets the injected `<script>` element as executable content instead of displaying it as plain text.

An attacker can therefore modify their profile and place a malicious JavaScript payload in the `lname` field. The payload may then execute whenever another user visits a page that displays the attacker's stored profile information.

Potentially affected locations may include:

* User profile pages
* User-management interfaces
* Task assignment pages
* Project member lists
* Administrative dashboards
* Activity or account listings

The exact execution location should be documented based on the page where the stored value was observed to execute.

Successful exploitation may allow an attacker to execute arbitrary JavaScript within the origin of the Task Management System in the browser of another authenticated user.

Depending on the victim's privileges and the application's security controls, this may allow unauthorized actions to be performed using the victim's active session, manipulation of displayed application content, access to information available to client-side scripts, or phishing and interface-redressing attacks.

The vulnerability results from inadequate handling of user-controlled profile data. User input should be validated according to the expected format and, critically, encoded according to the output context whenever it is rendered.

Because the injected payload is stored by the application and may execute during later requests, this issue should be classified as **Stored Cross-Site Scripting (Stored XSS)** if persistence and execution have been confirmed.

---

# Root Cause

The vulnerability is caused by improper handling of user-controlled profile information.

Potential contributing factors include:

* The `lname` parameter accepts arbitrary HTML or JavaScript.
* The supplied value is stored in the application's database.
* Stored profile information is later rendered without HTML output encoding.
* The application does not sufficiently restrict the expected format of name fields.
* User-controlled data is inserted directly into an HTML response.

A vulnerable implementation may resemble:

```php
$lname = $_POST['lname'];

$sql = "UPDATE users SET lname='$lname' WHERE id='$user_id'";
mysqli_query($conn, $sql);
```

Followed by unsafe output such as:

```php
echo $row['lname'];
```

The primary XSS issue occurs when the stored value is rendered without encoding.

The exact application source code should be reviewed and included in the final advisory where available.

---

# Affected Endpoint

```text
POST /PROJECT_NEW/user/UpdateUserProfile.php
```

---

# Vulnerable Parameter

```text
lname
```

---

# Additional Parameters

```text
fname
updateName
```

---

# Authentication Requirements

The supplied request contains a PHP session cookie, indicating that access to the affected profile-update functionality may require an authenticated user session.

Therefore, the likely attack requirements are:

```text
Privileges Required: Low
User Interaction: Required
```

The attacker must first be able to update a profile containing the malicious value, and another user must view a page where the value is rendered for the stored XSS payload to execute.

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
Cookie: PHPSESSID=8tap31h6vpj4vf3k0ekc0s72j9
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/PROJECT_NEW/user/UpdateUserProfile.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

fname=Anas++&lname=%20edas3%3cscript%3ealert(1)%3c%2fscript%3el47wi&updateName=
```

Decoded request body:

```text
fname=Anas++
lname= edas3<script>alert(1)</script>l47wi
updateName=
```

The injected JavaScript payload is:

```html
<script>alert(1)</script>
```

---

# Steps to Reproduce

1. Deploy the Task Management System in PHP in an authorized testing environment.

2. Authenticate using a valid user account.

3. Navigate to the user profile update page:

```text
/PROJECT_NEW/user/UpdateUserProfile.php
```

4. Intercept the profile update request using an HTTP testing proxy.

5. Modify the `lname` parameter so that it contains:

```html
<script>alert(1)</script>
```

6. Submit the modified request.

7. Navigate to a page where the updated user's last name is displayed.

8. Observe whether the browser executes the stored JavaScript payload.

9. Refresh the page or log in with another authorized testing account and revisit the affected page.

10. If the payload remains present and executes during subsequent requests, Stored XSS is confirmed.

---

# Expected Result

The application should treat the last-name value exclusively as user data.

A submitted value containing HTML should either be rejected according to the application's validation policy or safely displayed as text.

For example:

```text
<script>alert(1)</script>
```

should never execute as JavaScript.

---

# Observed Result

The supplied request places a JavaScript payload within the `lname` profile field.

If the application stores this value and executes it when the user's profile information is later displayed, this confirms a Stored Cross-Site Scripting vulnerability.

The final report should identify the exact page on which execution was observed.

---

# Impact

Successful exploitation may allow an authenticated attacker to execute arbitrary JavaScript in the browser of users who view the affected stored profile information.

Potential impact includes:

* Performing unauthorized actions using the victim's active session.
* Manipulating application content displayed to the victim.
* Accessing sensitive information exposed to client-side JavaScript.
* Creating deceptive application interfaces.
* Redirecting users to attacker-controlled content.
* Targeting privileged users who view profile or user-management pages.
* Persistently affecting multiple users until the malicious value is removed.

The practical severity depends heavily on which roles can view the affected profile data.

If an administrator is required to view the attacker's stored profile, the issue may represent an authenticated user-to-administrator Stored XSS attack.

---

# Vulnerability Classification

## CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

## OWASP Top 10

**A03:2021 – Injection**

---

# Remediation

## Apply Contextual Output Encoding

All user-controlled profile information should be properly encoded when rendered into HTML.

For example:

```php
echo htmlspecialchars(
    $row['lname'],
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

Output encoding should be applied according to the specific rendering context.

---

## Validate Name Fields

Because `fname` and `lname` represent names, the application should enforce reasonable:

* Length restrictions
* Character restrictions appropriate to supported languages
* Data-type validation

Input validation should be implemented server-side.

However, input validation must not replace output encoding.

---

## Use Prepared Statements

Although prepared statements do not prevent XSS, profile-update database operations should also use parameterized SQL queries to protect against SQL injection.

Example:

```php
$stmt = $conn->prepare(
    'UPDATE users SET fname = ?, lname = ? WHERE id = ?'
);

$stmt->bind_param(
    'ssi',
    $fname,
    $lname,
    $userId
);

$stmt->execute();
```

---

## Implement Content Security Policy

A restrictive Content Security Policy can reduce the impact of certain XSS vulnerabilities.

For example:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be treated as defense in depth and not as a replacement for correct output encoding.

---

## Review All Stored User Data

Developers should review every location where profile information is displayed, including:

* User profile pages
* Administrator panels
* Project pages
* Task assignment interfaces
* User lists
* Navigation elements

All user-controlled values should be encoded before being inserted into an HTML response.

---

# Conclusion

The Task Management System in PHP contains a potential Stored Cross-Site Scripting vulnerability in the `lname` parameter processed by `/PROJECT_NEW/user/UpdateUserProfile.php`.

An authenticated user can submit a last-name value containing JavaScript markup. If the supplied value is persisted in the application's database and subsequently rendered without contextual output encoding, the malicious script executes in the browser of users viewing the affected profile information.

If persistence and subsequent execution have been confirmed, the issue should be classified as **Stored Cross-Site Scripting under CWE-79**.

The vulnerability should be remediated by implementing contextual output encoding, enforcing appropriate server-side validation, reviewing all locations where stored profile information is displayed, and deploying defense-in-depth controls such as Content Security Policy.


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/01f92232-3043-4f00-97f6-2cdc9c3e8ba7" />
