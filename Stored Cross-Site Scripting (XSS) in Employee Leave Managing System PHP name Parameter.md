## Stored Cross-Site Scripting (XSS) in Employee Leave Managing System PHP name Parameter
## Credit

Discovered by: 
Ahmad Marzook

## Product

Employee Leave Managing System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/employee-leave-managing-system-in-php-with-source-code/

## Affected Version

Employee Leave Managing System in PHP v1.0

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

The Employee Leave Managing System in PHP v1.0 is vulnerable to a Stored Cross-Site Scripting (XSS) vulnerability in the employee profile update functionality.

## The vulnerability exists in the following endpoint:

/EmpManageSys/editaction.php

The application allows authenticated users to update employee information by submitting several parameters, including name, email, pos, and password. The value supplied in the name parameter is accepted and stored without adequate validation or sanitization.

When the stored employee information is subsequently displayed within the application, the name value is rendered directly into the HTML response without proper output encoding. Because user-controlled data is inserted into the page without neutralizing HTML or JavaScript, malicious code embedded in the input is interpreted and executed by the browser.

## For example, an attacker can submit the following payload in the name parameter:

<details/open/ontoggle=alert(1)>1

Once the update request is processed, the malicious payload is stored in the application's backend. Whenever the employee profile or any page displaying the employee name is viewed, the injected JavaScript executes automatically in the browser of the user accessing the page.

The vulnerability exists because the application:

Stores user-controlled input without sufficient validation.
Does not sanitize HTML content before saving it.
Fails to perform contextual output encoding when rendering stored values.

Successful exploitation allows attackers to execute arbitrary JavaScript in the context of authenticated users. Depending on the privileges of the victim, this may result in session hijacking, theft of sensitive information, execution of unauthorized actions, or injection of malicious content into the application interface.

Because the payload persists in the database and affects every user who views the compromised record, the vulnerability is classified as a Stored (Persistent) Cross-Site Scripting vulnerability.

## Root Cause

The vulnerability is caused by improper handling of user-supplied input.

Specifically:

No validation of the name parameter before storing it.
Lack of sanitization of HTML content.
Missing output encoding (htmlspecialchars() or equivalent) before rendering database content.
Direct insertion of database values into HTML pages.

A typical vulnerable implementation resembles:

$name = $_POST['name'];

mysqli_query($conn,
"UPDATE employee SET name='$name' WHERE id='$id'");

...

echo $row['name'];

Because the stored value is printed without HTML encoding, arbitrary JavaScript executes in the user's browser.
Affected Endpoint
 ```plain
POST /EmpManageSys/editaction.php
## Vulnerable Parameter
name
HTTP Method
POST
Proof of Concept
Full HTTP Request
POST /EmpManageSys/editaction.php HTTP/1.1
Host: localhost
Content-Length: 121
Cache-Control: max-age=0
sec-ch-ua: "Not-A.Brand";v="24", "Chromium";v="146"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Accept-Language: en-US,en;q=0.9
Origin: http://localhost
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://localhost/EmpManageSys/edit.php?id=32
Cookie: PHPSESSID=8sgapf310jnc0psdjc89q68h8e

name=%3Cdetails%2Fopen%2Fontoggle%3Dalert%281%29%3E1&
email=emma%40gmail.com&
pos=Artist1&
password=emma&
id=32&
update=Update
 ```
## Steps to Reproduce
Install the Employee Leave Managing System in PHP.
Authenticate as a user with permission to edit employee information.
Navigate to the employee edit page.
Intercept the update request using Burp Suite or another proxy.
Replace the value of the name parameter with:
<details/open/ontoggle=alert(1)>1
Submit the request.
Open the employee profile or any page displaying the updated employee name.
Result

The injected payload is stored in the application database.

Whenever the affected employee record is viewed, the browser executes the injected JavaScript, confirming the presence of a Stored Cross-Site Scripting vulnerability.

## Impact

An attacker who successfully exploits this vulnerability may be able to:

Execute arbitrary JavaScript in victim browsers.
Hijack authenticated user or administrator sessions.
Steal session cookies.
Perform actions on behalf of authenticated users.
Modify application content.
Inject phishing content or malicious scripts.
Access sensitive information available within the victim's session.

Because the payload is persistent, every user who views the affected employee record is exposed to the attack.

##  Vulnerability Classification

CWE

CWE-79 – Cross-Site Scripting (XSS)

OWASP Top 10

A03:2021 – Injection
Suggested Remediation
Output Encoding

Always encode user-controlled data before rendering it in HTML.

Example:

echo htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
Input Validation

Validate user input according to expected character sets and reject unexpected HTML or JavaScript content.

Content Security Policy (CSP)

Deploy a restrictive Content Security Policy to reduce the impact of injected scripts.

Example:

Content-Security-Policy:
default-src 'self';
script-src 'self';
object-src 'none';

<img width="1483" height="913" alt="Image" src="https://github.com/user-attachments/assets/eb2fc0ff-17a8-4b55-b816-19c8dae4057b" />
