## Stored Cross-Site Scripting (XSS) in Simple Inventory System PHP last_name Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Simple Inventory System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/simple-inventory-system-in-php-with-source-code/

## Affected Version

Simple Inventory System in PHP v1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

OWASP Top 10

A03:2021 – Injection

## Severity

Medium

CVSS v3.1 Score

6.5 (Medium)

Vector

CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N
Description

The Simple Inventory System in PHP v1.0 is vulnerable to a Stored Cross-Site Scripting (XSS) vulnerability in the user registration functionality.

## The vulnerability exists in the following endpoint:

/InventoryManagement/register.php

During the registration process, the application accepts user-controlled input through multiple parameters, including first_name, last_name, username, email, and mobile. The value supplied in the last_name parameter is stored by the application without sufficient validation or sanitization.

When the registered user's information is later displayed within the application, the stored value is rendered directly into the HTML response without applying proper output encoding. Because user-controlled input is inserted into the page without being safely encoded, malicious HTML or JavaScript supplied by an attacker is interpreted and executed by the browser.

For example, an attacker can submit the following payload in the last_name parameter:

<script>alert(1)</script>

After the registration request is processed, the payload is stored in the application's database. Whenever an administrator or another authenticated user views the affected user profile, user list, or any page displaying the stored value, the injected JavaScript executes automatically.

The vulnerability exists because the application stores user input without adequate sanitization and later renders the stored data without contextual output encoding.

## This issue is caused by:

Lack of server-side validation of user-controlled input.
Failure to sanitize HTML or JavaScript before storing data.
Missing output encoding when displaying stored values.
Direct rendering of database content within HTML pages.

Successful exploitation allows an attacker to execute arbitrary JavaScript in the context of authenticated users. Depending on the privileges of the victim, this may result in session hijacking, theft of authentication cookies, account takeover, execution of unauthorized actions, modification of application content, or delivery of phishing interfaces.

Because the injected payload is permanently stored and executes whenever the affected record is viewed, the vulnerability is classified as a Stored (Persistent) Cross-Site Scripting (XSS) vulnerability.

## Root Cause

The vulnerability occurs because user-controlled input supplied through the last_name parameter is stored without proper validation or sanitization and is later rendered without output encoding.

Typical vulnerable code resembles:

$lastname = $_POST['last_name'];

mysqli_query($conn,
"INSERT INTO users(first_name,last_name,username,email)
VALUES('$firstname','$lastname','$username','$email')");

...

echo $row['last_name'];

Since the stored value is printed directly into the HTML response, browsers interpret injected HTML and JavaScript as executable content.

## Affected Endpoint
POST /InventoryManagement/register.php
## Vulnerable Parameter
last_name
## HTTP Method
POST
## Proof of Concept
Full HTTP Request
 ```plain
POST /InventoryManagement/register.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="146", "Not=A?Brand";v="8", "Chromium";v="146"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Origin: https://localhost
Referer: https://localhost/InventoryManagement/register.php
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=r4fm6nv8ub48ikv21hbjkgmg09

first_name=aaafd&
last_name=aaafdwv8fa<script>alert(1)</script>c1u7v&
username=aaafd&
email=aaafd@burpcollaborator.net&
mobile=aaafd&
password_1=123456&
password_2=123456&
reg_user=Register
 ```
## Steps to Reproduce
Install the Simple Inventory System in PHP.
Navigate to the registration page.
Intercept the registration request using Burp Suite.
Replace the last_name parameter with:
<script>alert(1)</script>
Submit the registration request.
Log in as an administrator or navigate to the page displaying registered users.
Observe that the injected JavaScript executes automatically.
Result

The malicious payload is stored in the application's database and executes when the affected user information is viewed, confirming the presence of a Stored Cross-Site Scripting vulnerability.

## Impact

Successful exploitation allows attackers to:

Execute arbitrary JavaScript in victim browsers.
Hijack authenticated administrator sessions.
Steal authentication cookies.
Perform actions on behalf of authenticated users.
Modify application content.
Inject phishing pages.
Access sensitive information available within the victim's session.

Because the payload is persistent, every user who views the affected record may be impacted.

Vulnerability Classification
CWE
CWE-79 – Cross-Site Scripting (XSS)
OWASP Top 10
A03:2021 – Injection
Suggested Remediation
Output Encoding

Encode user-controlled data before rendering it:

echo htmlspecialchars($last_name, ENT_QUOTES, 'UTF-8');
Input Validation

Validate user input against expected formats and reject HTML or JavaScript where it is not required.

Content Security Policy (CSP)

Implement a restrictive Content Security Policy to reduce the impact of injected scripts.

Example:

Content-Security-Policy: default-src 'self'; script-src 'self';


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/21a91868-0f01-48b9-8569-018762576930" />
