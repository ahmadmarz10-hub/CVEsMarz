## Stored Cross-Site Scripting (XSS) in Online Shopping System PHP email Parameter
## Credit

## Discovered by: Ahmad Marzook

## Product

Online Shopping System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-shopping-system-in-php-with-source-code/

## Affected Version

Online Shopping System in PHP v1.0

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

CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N
Description

The Online Shopping System in PHP v1.0 is vulnerable to a Stored Cross-Site Scripting (XSS) vulnerability in the newsletter subscription functionality.

## The vulnerability exists in the following endpoint:

/online-shopping-system/offersmail.php

The application allows visitors to subscribe to promotional offers by submitting an email address through the email parameter. The supplied value is accepted and stored without adequate validation or sanitization.

When the stored email address is later displayed within the administrative interface or any page that lists newsletter subscribers, the application renders the stored value directly into the HTML response without applying proper output encoding.

Because user-controlled input is inserted into the HTML page without being safely encoded, an attacker can inject arbitrary HTML or JavaScript into the email parameter. For example, an attacker can submit the following payload:

<script>alert(1)</script>

After the subscription request is processed, the malicious payload is stored by the application. Whenever an administrator or another privileged user views the subscriber list or another page displaying the stored email address, the injected JavaScript executes automatically in the victim's browser.

The vulnerability exists because the application fails to validate or sanitize user-controlled input before storing it and does not apply contextual output encoding when rendering stored values.

Successful exploitation allows attackers to execute arbitrary JavaScript in the context of authenticated users. Depending on the privileges of the victim, this may result in session hijacking, theft of authentication cookies, unauthorized actions performed on behalf of administrators, modification of application content, or delivery of phishing interfaces.

Because the malicious payload remains stored in the application's database and is executed whenever the affected record is viewed, the issue is classified as a Stored (Persistent) Cross-Site Scripting (XSS) vulnerability.

## Root Cause

The vulnerability is caused by improper handling of user-supplied input.

Specifically:

The email parameter accepts arbitrary user input.
User input is stored without adequate validation or sanitization.
Stored values are rendered without HTML output encoding.
The application directly inserts database values into HTML pages.

A typical vulnerable implementation resembles:

$email = $_POST['email'];

mysqli_query($conn,
"INSERT INTO subscribers(email) VALUES('$email')");

...

echo $row['email'];

Because the stored value is rendered directly into the HTML response, malicious HTML or JavaScript is interpreted and executed by the browser.

## Affected Endpoint
POST /online-shopping-system/offersmail.php
## Vulnerable Parameter
email
## HTTP Method
POST
## Proof of Concept
 ```plain
Full HTTP Request
POST /online-shopping-system/offersmail.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="146", "Not=A?Brand";v="8", "Chromium";v="146"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: */*
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=i7s4bfr66udpck4v6um1v7rddb
Origin: https://localhost
X-Requested-With: XMLHttpRequest
Referer: https://localhost/online-shopping-system/
Content-Type: application/x-www-form-urlencoded; charset=UTF-8

email=j4sap<script>alert(1)</script>pxgp5
 ```

## Steps to Reproduce
Install the Online Shopping System in PHP.
Navigate to the homepage.
Locate the newsletter subscription form.
Intercept the subscription request using Burp Suite.
Replace the value of the email parameter with:
<script>alert(1)</script>
Submit the request.
Access the administrative interface or any page displaying newsletter subscribers.
Observe that the injected JavaScript executes automatically.
Result

The payload is successfully stored in the backend database and executes when the stored email address is viewed, confirming the presence of a Stored Cross-Site Scripting vulnerability.

## Impact

Successful exploitation allows attackers to:

Execute arbitrary JavaScript in administrator browsers.
Hijack authenticated sessions.
Steal authentication cookies.
Perform unauthorized actions on behalf of administrators.
Modify application content.
Inject phishing interfaces.
Access sensitive information available within the victim's session.

Because the payload is persistent, all privileged users who view the affected subscriber record may be impacted.

Vulnerability Classification
CWE
CWE-79 – Cross-Site Scripting (XSS)
OWASP Top 10
A03:2021 – Injection
Suggested Remediation
Output Encoding

Always encode user-controlled data before rendering it in HTML.

Example:

echo htmlspecialchars($email, ENT_QUOTES, 'UTF-8');
Input Validation

Validate email addresses using server-side validation (e.g., filter_var($email, FILTER_VALIDATE_EMAIL)) and reject malformed input.

Content Security Policy (CSP)

Deploy a restrictive Content Security Policy to reduce the impact of injected scripts.

Example:

Content-Security-Policy: default-src 'self'; script-src 'self';


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/ea602776-5071-497f-a8ff-51c4830ca647" />
