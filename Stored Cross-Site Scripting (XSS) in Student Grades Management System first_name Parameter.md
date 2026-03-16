## Stored Cross-Site Scripting (XSS) in Student Grades Management System first_name Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Student Grades Management System

## Vendor

SourceCodester

## Vendor Homepage

https://www.sourcecodester.com/php/18408/student-grades-management-system-using-html-css-and-javascript-source-code.html

## Affected Version

Student Grades Management System v1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in the administrative user management functionality of the Student Grades Management System.

The vulnerability occurs when the application processes the first_name parameter during the user creation process.

## The affected endpoint is:

/student-grades-management-system/admin.php

During user creation, the application accepts several parameters through an HTTP POST request, including:

first_name

last_name

username

email

role

password

The value supplied in the first_name parameter is stored in the backend database without proper input validation or sanitization.

Because the stored value is later rendered in the application interface without applying output encoding, malicious HTML or JavaScript code can be executed in the browser of users who view the affected page.

An attacker can exploit this issue by submitting specially crafted HTML or JavaScript code in the first_name field during the user creation process.

Once stored in the database, the injected payload becomes persistent and executes automatically when the affected user information is displayed within the administrative interface.

Example payload used during testing:

<details/open/ontoggle=prompt(origin)>

When the page displaying the stored user information is accessed, the injected payload executes in the browser of the victim user.

Because the payload is stored server-side and executed whenever the affected content is viewed, this vulnerability is classified as a Stored (Persistent) Cross-Site Scripting vulnerability.

## Root Cause

The vulnerability occurs due to improper handling of user-controlled input.

The application stores the value of the first_name parameter directly in the database without performing input validation or sanitization. When the stored value is later rendered in the HTML interface, it is displayed without applying output encoding functions such as htmlspecialchars().

This results in user-supplied HTML being interpreted by the browser instead of being treated as plain text.

The combination of unsanitized input storage and unescaped output rendering leads to the stored XSS vulnerability.

## Affected Endpoint
/student-grades-management-system/admin.php
## Vulnerable Parameter
first_name
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
```plain
POST /student-grades-management-system/admin.php HTTP/1.1
Host: localhost
Cookie: PHPSESSID=mfidnvei6c1pgdfuibo14p9tlk
Content-Length: 146
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="145", "Not:A-Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "macOS"
Accept-Language: en-US,en;q=0.9
Origin: https://localhost
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://localhost/student-grades-management-system/admin.php
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive

first_name=%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E&last_name=1&username=1&email=admin%40example.com&role=student&password=1&add_user=
```
PoC 1

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/679d6a68-86e1-450a-9503-2d87b5b85e87" />

## Steps to Reproduce

Install the Student Grades Management System application.

Log in to the administrative panel.

Navigate to the user management page.

Intercept the request for adding a new user using a proxy tool such as Burp Suite.

Insert the following payload into the first_name parameter:

<details/open/ontoggle=prompt(origin)>

Submit the modified request.

Navigate to the page displaying the created user.

## Result

When the stored user information is displayed, the injected payload executes in the browser.

The JavaScript prompt confirms that the application renders stored user input without proper sanitization.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary JavaScript in victim browsers

Steal authentication cookies

Hijack administrator sessions

Perform unauthorized actions within the application

Inject malicious content into the administrative interface

Conduct phishing attacks against users

Because the payload is stored in the database, the attack may affect all users who view the compromised page.

## Vulnerability Classification

CWE
CWE-79 – Cross-Site Scripting

OWASP Top 10
A03:2021 – Injection

Suggested Remediation
Output Encoding

Encode user-controlled data before rendering it in HTML.

Example:

echo htmlspecialchars($row['first_name'], ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize user input before storing it in the database.

Reject potentially dangerous HTML or script content.

Content Security Policy

Implement a Content Security Policy (CSP) to reduce the risk of malicious script execution.

Example:

Content-Security-Policy: default-src 'self'

PoC 2 

https://github.com/user-attachments/assets/efe9819f-458b-4362-82d1-bec3e5b63dc0
