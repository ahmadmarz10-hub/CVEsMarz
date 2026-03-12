## Stored Cross-Site Scripting (XSS) in Blood Bank Managing System PHP statename Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Blood Bank Managing System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/bloodbank-managing-system-in-php-with-source-code/

## Affected Version

Blood Bank Managing System v1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Blood Bank Managing System in PHP within the state management functionality.

The vulnerability occurs in the administrative component responsible for adding new states to the system. The affected endpoint processes user input submitted through the statename parameter.

During the state creation process, the application accepts the value of statename from an HTTP POST request and stores it directly in the backend database without performing proper input validation or sanitization.

Because the stored value is later rendered in the application interface without applying output encoding, malicious HTML or JavaScript code may execute in the browser of users who view the affected page.

This behavior allows attackers to inject arbitrary scripts into the application by submitting specially crafted input when creating a new state entry.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input.

The application retrieves the value of the statename parameter directly from the HTTP request and stores it in the database without sanitization. When the stored value is displayed in the web interface, it is rendered without applying HTML escaping functions such as htmlspecialchars().

Because of this, HTML elements embedded in the stored value are interpreted by the browser instead of being treated as plain text.

This combination of unsanitized input storage and unescaped output rendering results in a Stored Cross-Site Scripting vulnerability.

## Affected Endpoint
/Blood_Bank/admin_state.php
## Vulnerable Parameter
statename
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
```plain
POST /Blood_Bank/admin_state.php HTTP/1.1
Host: localhost
Content-Length: 96
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
Referer: http://localhost/Blood_Bank/admin_state.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=r8lic3o19qngid0clllb39s0n6
Connection: keep-alive

COUNTRY=13&statename=%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E&state_submit=Add+State
```
## PoC 1

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/0d5efd22-8580-42c5-9bb4-1e26524f9561" />

## Steps to Reproduce

Install the Blood Bank Managing System in PHP.

Login to the administrative panel.

Navigate to the State Management page.

Intercept the request when adding a new state using a proxy tool such as Burp Suite.

Insert the following payload into the statename parameter:

<details/open/ontoggle=prompt(origin)>

Submit the modified request.

Open the page where the state list is displayed.

## Result

When the affected page is viewed, the injected payload executes in the browser of the user viewing the page.

The JavaScript prompt confirms that the application renders stored user input without proper sanitization.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary JavaScript in victim browsers

Steal session cookies

Hijack administrator sessions

Perform unauthorized actions in the application

Inject malicious content into the administrative interface

Conduct phishing attacks against application users

Because the payload is stored in the database, the attack may affect all users who view the compromised page.

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation

Developers should implement the following security measures.

Output Encoding

Encode user-controlled data before rendering it in HTML responses.

Example:

echo htmlspecialchars($row['statename'], ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize user input before storing it in the database.

Reject or filter suspicious HTML or script content.

Content Security Policy

Implement a Content Security Policy (CSP) to mitigate script execution risks.

Example:

Content-Security-Policy: default-src 'self'

## PoC 2

https://github.com/user-attachments/assets/ce655b56-78f1-4d2b-8b20-2432b8c6830d
