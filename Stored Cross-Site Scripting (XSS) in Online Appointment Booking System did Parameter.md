## Stored Cross-Site Scripting (XSS) in Online Appointment Booking System did Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Online Appointment Booking System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-appointment-booking-system-in-php-css-javascript-and-mysql-free-download/

## Affected Version

Online Appointment Booking System v1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Online Appointment Booking System in PHP within the administrative doctor management functionality.

The vulnerability occurs in the following component:

/Online-Appointment-Booking-System-master/Admin/deletedoctor.php

The application processes user input submitted via HTTP POST parameters including:

did (doctor ID)

doctorname

During request handling, the application accepts user-controlled input from these parameters and uses it without proper validation or sanitization.

Testing indicates that malicious HTML or JavaScript code injected into the did parameter can be processed and later rendered within the application interface without output encoding.

Because the application fails to properly sanitize or encode the stored or reflected values, attacker-controlled payloads can execute in the browser of users interacting with the affected functionality.

Example payload used during testing:

<script>alert(1)</script>

Once the malicious input is processed, it may be reflected or stored and executed when the affected page is loaded or when application responses include the injected data.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input.

The application retrieves the value of the did parameter directly from the HTTP request and uses it without validation, sanitization, or encoding. When the value is later included in HTML responses, it is rendered without applying output encoding functions such as htmlspecialchars().

This allows HTML and JavaScript content embedded in user input to be interpreted by the browser.

The lack of input validation combined with unescaped output leads to a Cross-Site Scripting vulnerability.

## Affected Endpoint
/Online-Appointment-Booking-System-master/Admin/deletedoctor.php
## Vulnerable Parameter
did
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
 ```plain
POST /Online-Appointment-Booking-System-master/Admin/deletedoctor.php HTTP/1.1
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
Cookie: PHPSESSID=bcr4mq8tcag2k2i9b8tjvhcekg
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/Online-Appointment-Booking-System-master/Admin/deletedoctor.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 76

did=%7d%7dw281u%3cscript%3ealert(1)%3c%2fscript%3eq8f58&Submit1=&doctorname=
 ```
## PoC 1

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/99247582-e37a-4bcc-926e-178e857a3a0b" />

## Steps to Reproduce

Install the Online Appointment Booking System in PHP.

Login to the administrative panel.

Navigate to the doctor management or delete doctor functionality.

Intercept the request using a proxy tool such as Burp Suite.

Insert the following payload into the did parameter:

<script>alert(1)</script>

Submit the modified request.

Observe the application response or subsequent page loads.

## Result

The injected payload is processed and executed in the browser, demonstrating that user input is not properly sanitized or encoded.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary JavaScript in victim browsers

Hijack administrator sessions

Steal authentication cookies

Perform unauthorized administrative actions

Inject malicious content into the application interface

Conduct phishing attacks

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

Suggested Remediation
Output Encoding

Encode user-controlled data before rendering it in HTML.

Example:

echo htmlspecialchars($value, ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize all user inputs.

Reject unexpected or malicious content.

Use Prepared Statements

Avoid inserting user input directly into SQL queries.

Content Security Policy

Implement CSP headers:

Content-Security-Policy: default-src 'self'

## PoC 2 


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/ca9c0627-a549-4e7a-af5b-412a5c9568b4" />
