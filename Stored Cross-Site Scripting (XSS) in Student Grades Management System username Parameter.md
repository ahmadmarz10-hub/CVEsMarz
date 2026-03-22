## Stored Cross-Site Scripting (XSS) in Student Grades Management System username Parameter
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

The vulnerability occurs when the application processes user input supplied through the username parameter during the user creation process.

## The affected endpoint is:

/student-grades-management-system/admin.php

During user creation, the application accepts several parameters via an HTTP POST request, including:

first_name

last_name

username

email

role

password

The value provided in the username parameter is stored in the backend database without proper validation or sanitization.

Because the stored value is later rendered within the application interface without applying output encoding, malicious HTML or JavaScript code can be executed in the browser of users who view the affected data.

An attacker can exploit this vulnerability by submitting specially crafted input in the username field. Once stored, the payload becomes persistent and executes when the application displays the stored username.

## payload used during testing:

<script>alert(1)</script>

In the captured request, the payload was injected as part of the username value:

username=admingdkxf<script>alert(1)</script>s6zk6

When the affected page is loaded, the injected script executes in the browser.

Because the payload is stored server-side and executed whenever the affected data is rendered, this issue is classified as a Stored (Persistent) Cross-Site Scripting vulnerability.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input.

The application directly stores the username parameter from the HTTP request into the database without performing input validation or sanitization. When the stored value is later rendered in the HTML interface, it is output without encoding using functions such as htmlspecialchars().

This allows HTML and JavaScript content embedded in user input to be interpreted and executed by the browser.

The combination of unsanitized input storage and unescaped output rendering leads to the Stored XSS vulnerability.

## Affected Endpoint
/student-grades-management-system/admin.php
## Vulnerable Parameter
username
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
 ```plain
POST /student-grades-management-system/admin.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="145", "Not=A?Brand";v="8", "Chromium";v="145"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=tr02kit0rvsmkabun19sn795pm
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/student-grades-management-system/admin.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 121

first_name=admin&last_name=admin&username=admingdkxf%3cscript%3ealert(1)%3c%2fscript%3es6zk6&email=admin%40burpcollaborator.net&role=admin&password=admin123&add_user=
 ```
## Steps to Reproduce

Install the Student Grades Management System.

Login to the administrative panel.

Navigate to the user management section.

Intercept the user creation request using a proxy tool such as Burp Suite.

Insert the following payload into the username parameter:

<script>alert(1)</script>

Submit the modified request.

Navigate to the page displaying user accounts.

## Result

When the stored user data is displayed, the injected JavaScript payload executes in the browser.

This confirms that the application renders stored user input without proper sanitization or encoding.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary JavaScript in victim browsers

Steal session cookies

Hijack administrator sessions

Perform unauthorized actions within the application

Inject malicious content into the administrative interface

Conduct phishing attacks

Because the payload is stored in the database, the attack affects all users who view the compromised data.

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

Suggested Remediation
Output Encoding

Encode user-controlled data before rendering it in HTML:

echo htmlspecialchars($row['username'], ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize user input before storing it in the database.

Content Security Policy

Implement CSP headers:

Content-Security-Policy: default-src 'self'

## PoC 

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/7aa5f1f0-9199-41d9-b4ca-09f49fd8cee8" />
