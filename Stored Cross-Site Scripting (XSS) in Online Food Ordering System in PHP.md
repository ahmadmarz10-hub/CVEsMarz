## Stored Cross-Site Scripting (XSS) in Online Food Ordering System in PHP name Parameter

## Credit
Discovered by:
Ahmad Marzook

## Product

Online Food Ordering System in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/online-food-ordering-system-in-php-with-source-code/

## Affected Version

1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

The Online Food Ordering System in PHP 1.0 is vulnerable to a Stored Cross-Site Scripting (XSS) vulnerability in the name parameter processed by the /dbfood/contact.php endpoint. The vulnerability exists because the application fails to properly sanitize or encode user-supplied input before storing it in the backend database and rendering it within the web application interface.

The contact form functionality allows users to submit information such as name, email, phone, and message content. The value provided in the name field is stored by the application and later displayed within administrative pages or other application views without proper output encoding.

An attacker can exploit this vulnerability by injecting malicious JavaScript code into the name parameter. Because the application stores the input and later renders it without sanitization, the injected script executes automatically whenever the stored contact message is viewed.

For example, an attacker can submit the payload <details/open/ontoggle=prompt(origin)> within the name parameter. Once stored in the database, this payload executes in the browser of administrators or other users who view the contact messages, demonstrating a persistent XSS vulnerability.

Successful exploitation allows attackers to execute arbitrary JavaScript in the context of the victim’s browser session. This may result in session hijacking, theft of authentication cookies, unauthorized actions performed on behalf of authenticated users, or the injection of malicious content into the application interface.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input. The application accepts user input from the contact form and stores it directly in the database without performing sufficient validation, sanitization, or encoding.

When the stored data is later displayed within HTML pages, the application fails to apply proper output encoding functions such as htmlspecialchars() or equivalent filtering mechanisms. As a result, malicious HTML or JavaScript code injected by an attacker is interpreted and executed by the user's browser.

This issue occurs due to:

Lack of input validation for user-submitted fields

Absence of output encoding before rendering user data in HTML

Direct rendering of stored user-controlled data in the application interface

## Affected Endpoint
/dbfood/contact.php
## Vulnerable Parameter
name
## Proof of Concept

The vulnerability can be triggered using the following HTTP request:
 ```plain
POST /dbfood/contact.php HTTP/1.1
Host: localhost
Content-Length: 111
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Accept-Language: en-US,en;q=0.9
Origin: http://localhost
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost/dbfood/contact.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=r8lic3o19qngid0clllb39s0n6
Connection: keep-alive

name=%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E&email=admin%40example.com&phone=&msgtxt=scsc&message=
 ```

## Encoded payload used:

%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E

## Steps to Reproduce

Install and run the Online Food Ordering System in PHP application.

Navigate to the Contact page.

Submit the malicious payload in the name parameter.

Save the contact message.

Access the administrative interface or page where contact messages are displayed.

Observe that the injected JavaScript executes automatically.

## Result

When the stored contact message is viewed, the injected JavaScript payload executes automatically in the browser of the user viewing the message, confirming the presence of a Stored Cross-Site Scripting vulnerability.

## Impact

An attacker exploiting this vulnerability may be able to:

Execute arbitrary JavaScript in victim browsers

Hijack authenticated user sessions

Steal session cookies

Perform unauthorized actions on behalf of administrators

Inject malicious scripts into the application interface

Because the payload is stored in the database, the attack can affect all users who view the stored data.

## Remediation

Developers should implement the following security measures:

Output Encoding

Escape user-controlled input before rendering it in HTML responses.

## Example:

echo htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize all user input before storing it in the database.

Content Security Policy (CSP)

Implement a Content Security Policy to reduce the risk of script execution.

Security Testing

Conduct regular security testing to detect and mitigate injection vulnerabilities.

## Vulnerability Classification

CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

PoC

https://github.com/user-attachments/assets/e9fd9c0b-94be-4567-84f8-e9c2eff9272a
