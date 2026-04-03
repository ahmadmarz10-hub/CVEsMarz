## SQL Injection in Simple Laundry System PHP searchServiceId Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Simple Laundry System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/simple-laundry-system-in-php-with-source-code/

## Affected Version

Simple Laundry System v1.0

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

A SQL Injection vulnerability exists in the Simple Laundry System in PHP within the service tracking functionality.

The vulnerability is located in the following endpoint:

/Laundry_system/searchguest.php

The application processes user input through the searchServiceId parameter submitted via an HTTP POST request. This parameter is directly incorporated into SQL queries without proper validation, sanitization, or parameterized query handling.

Because the application fails to neutralize special SQL characters, attackers can inject malicious SQL code into the query. The supplied payload:

searchServiceId='

demonstrates that the input is not properly handled and may break the SQL query structure, indicating injectable behavior.

In typical implementations, this parameter is used in queries similar to:

$serviceId = $_POST['searchServiceId'];
$query = "SELECT * FROM services WHERE service_id = '$serviceId'";

Since the input is concatenated directly into the SQL statement, an attacker can manipulate the query logic by injecting arbitrary SQL expressions.

This vulnerability allows attackers to execute unauthorized SQL queries against the backend database.

## Root Cause

The vulnerability is caused by improper handling of user-controlled input.

Specifically:

User input is directly embedded into SQL queries

No input validation or sanitization is performed

Prepared statements and parameter binding are not used

The application trusts user input without verification

This leads to SQL injection.

## Affected Endpoint
/Laundry_system/searchguest.php
## Vulnerable Parameter
searchServiceId
## HTTP Method
POST
## Proof of Concept
Request
 ```plain
POST /Laundry_system/searchguest.php HTTP/1.1
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
Cookie: PHPSESSID=5u384qnj9gvgp4o6ck3rkaejsg
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/Laundry_system/trackservice.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 46

searchServiceId='&searchServiceIdSubmit=submit
 ```
## Steps to Reproduce

Install the Simple Laundry System in PHP.

Navigate to the service tracking page.

Intercept the request using Burp Suite.

Modify the searchServiceId parameter:

' OR 1=1--

Send the request.

Result

The application processes the injected payload, potentially returning unintended results or exposing database data, confirming SQL injection.

Impact

Successful exploitation allows attackers to:

Extract sensitive database information

Bypass application logic

Retrieve all records from the database

Modify or delete data

Execute arbitrary SQL queries

Potentially gain full control over the database

Vulnerability Classification
CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

Suggested Remediation
Use Prepared Statements
$stmt = $conn->prepare("SELECT * FROM services WHERE service_id = ?");
$stmt->execute([$serviceId]);
Input Validation

Validate input format (e.g., numeric IDs only).

Least Privilege

Use restricted database user permissions.

Security Testing

Conduct regular penetration testing and code audits.

## PoC

<img width="1508" height="877" alt="Image" src="https://github.com/user-attachments/assets/63790e30-500a-4823-9086-8b74f1cee107" />
