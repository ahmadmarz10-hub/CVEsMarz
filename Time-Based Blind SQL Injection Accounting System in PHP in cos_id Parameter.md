Time-Based Blind SQL Injection Accounting System in PHP in cos_id Parameter

Credit

Discovered by:
Ahmad Marzook


Product

Accounting System in PHP

Vendor

Code-Projects

Vendor URL

https://code-projects.org/accounting-system-in-php-with-source-code/

Affected Version

1.0

Vulnerability Type

SQL Injection (Time-Based Blind)

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command

Severity

High

Description

The Accounting System in PHP 1.0 is vulnerable to a Time-Based Blind SQL Injection vulnerability in the cos_id parameter of the /my_account/delete.php endpoint. The application fails to properly validate and sanitize user-supplied input before including it in SQL queries executed by the backend database.

The cos_id parameter is used to identify and delete customer records. However, the application directly incorporates this parameter into an SQL query without proper input validation or the use of prepared statements. This allows attackers to inject malicious SQL commands into the query.

An attacker can exploit this vulnerability by supplying specially crafted SQL payloads within the cos_id parameter. By leveraging database delay functions such as SLEEP(), an attacker can cause the server to delay its response for a measurable period of time. This confirms that the injected SQL statements are executed by the backend database.

Because the application does not display database error messages, attackers can exploit this vulnerability using a time-based blind SQL injection technique, where information about the database is inferred based on differences in server response time.

Successful exploitation may allow attackers to enumerate database structures, extract sensitive information such as user or administrative data, manipulate database records, or perform other unauthorized database operations depending on the privileges of the database account used by the application.

The vulnerability exists due to insufficient input validation and the absence of parameterized SQL queries when handling user-controlled input.

Affected Endpoint
/my_account/delete.php
Vulnerable Parameter
cos_id
Proof of Concept

GET /my_account/delete.php?cos_id=(select(0)from(select(sleep(15)))v)%2f%2a'%2B(select(0)from(select(sleep(15)))v)%2B'%22%2B(select(0)from(select(sleep(15)))v)%2B%22%2a%2f HTTP/1.1

Host: localhost

sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"

sec-ch-ua-mobile: ?0

sec-ch-ua-platform: "macOS"

Accept-Language: en-US,en;q=0.9

Upgrade-Insecure-Requests: 1

User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7

Sec-Fetch-Site: same-origin

Sec-Fetch-Mode: navigate

Sec-Fetch-User: ?1

Sec-Fetch-Dest: document

Accept-Encoding: gzip, deflate, br

Connection: keep-alive


Steps to Reproduce

Install and run the Accounting System in PHP application.

Navigate to the customer management functionality.

Intercept a request sent to /my_account/delete.php.

Modify the cos_id parameter with a time-based SQL injection payload.

Send the request to the server.

Observe the server response time.

Result

The server response is delayed by approximately 15 seconds, confirming that the injected SQL query containing the SLEEP(15) function is executed by the backend database.

This demonstrates a Time-Based Blind SQL Injection vulnerability.

Impact

An attacker exploiting this vulnerability may be able to:

Extract sensitive database information

Enumerate database tables and columns

Retrieve administrator or user credentials

Modify or delete database records

Potentially gain administrative access to the system

In severe scenarios, attackers could fully compromise the application's database.

Remediation

Developers should implement the following security measures:

Use Prepared Statements

Example:

$stmt = $pdo->prepare("DELETE FROM customers WHERE cos_id = ?");
$stmt->execute([$cos_id]);
Input Validation

Validate and sanitize all user-supplied input.

Least Privilege Principle

Use database accounts with minimal privileges.

Security Monitoring

Implement Web Application Firewalls (WAF) and regular security testing.

Vulnerability Classification

CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/418b0ace-3012-48a0-9ca9-9f5c47be0899" />
