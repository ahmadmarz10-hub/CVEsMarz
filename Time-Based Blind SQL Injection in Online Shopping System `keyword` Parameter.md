# Time-Based Blind SQL Injection in Online Shopping System `keyword` Parameter

## Credit

**Discovered by:** Ahmad Marzook

## Product

Online Shopping System in PHP

## Vendor

Code-Projects

## Vendor Homepage

`https://code-projects.org/online-shopping-system-in-php-with-source-code/`

## Affected Version

Online Shopping System in PHP, version unspecified.

The exact affected version should be confirmed from the downloaded package or application documentation before submitting the report.

## Vulnerability Type

Time-Based Blind SQL Injection

## CWE

**CWE-89 — Improper Neutralization of Special Elements Used in an SQL Command**

## OWASP Classification

**OWASP Top 10 A03:2021 — Injection**

## Severity

**High**

## Suggested CVSS v3.1 Score

**8.2 — High**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N
```

The score assumes that the search functionality is available without authentication. The privileges-required metric should be changed if authentication is necessary.

## Summary

The Online Shopping System in PHP contains a time-based blind SQL injection vulnerability in the product-search functionality.

The issue affects the `keyword` parameter processed by:

```text
POST /online-shopping-system/action.php
```

The application appears to incorporate the supplied search keyword into an SQL statement without securely separating user-controlled data from the SQL command. An attacker can therefore submit specially crafted SQL syntax that alters the structure of the backend query.

The submitted test value contains a conditional database delay using the MySQL `SLEEP()` function. A repeatable delay in the server response indicates that the injected expression was evaluated by the database, demonstrating that user-controlled input can influence SQL execution.

## Description

The Online Shopping System in PHP is affected by a **time-based blind SQL injection vulnerability** in the search functionality implemented by the `action.php` endpoint.

The vulnerable input is supplied through the `keyword` POST parameter while the `search` parameter is set to `1`. The application does not appear to use a parameterized SQL statement when processing this value. Consequently, SQL metacharacters and expressions included in the search keyword may be interpreted as part of the application's database query rather than treated exclusively as search data.

The provided request injects an SQL expression containing the MySQL `SLEEP(20)` function. When the database evaluates the injected expression, execution is intentionally delayed for approximately 20 seconds. Because blind SQL injection may not return database errors or query results directly in the HTTP response, the observable response-time difference acts as a side channel confirming whether an injected condition was executed.

An attacker could use repeated true-or-false conditions and response timing to infer information from the database one value at a time. Depending on the database account's permissions and the data available to the application, this could expose user records, administrator information, password hashes, customer details, order data, product information, and the internal database structure.

The finding should be considered confirmed only when the delay is repeatable and is absent from an equivalent control request that does not contain the database delay expression. Network instability, application processing time, or server load should be ruled out before public disclosure.

## Affected Endpoint

```text
POST /online-shopping-system/action.php
```

## Vulnerable Parameter

```text
keyword
```

## Supporting Parameter

```text
search=1
```

## Root Cause

The vulnerability is caused by constructing an SQL query using untrusted request data without parameterized queries.

A vulnerable implementation may resemble:

```php
$keyword = $_POST['keyword'];

$query = "SELECT * FROM products
          WHERE product_title LIKE '%" . $keyword . "%'";

$result = mysqli_query($connection, $query);
```

In this pattern, the `keyword` value is concatenated directly into the SQL statement. An attacker can terminate or modify the intended string context and introduce additional SQL expressions.

The exact vulnerable source code should be reviewed and quoted in the final advisory where available.

## Proof of Concept

The following request was supplied as evidence of the issue:

```http
POST /online-shopping-system/action.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="146", "Not=A?Brand";v="8", "Chromium";v="146"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: */*
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=7rvncohihrqcibkjt78i56vi9v
Origin: https://localhost
X-Requested-With: XMLHttpRequest
Referer: https://localhost/online-shopping-system/
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Content-Length: 75

search=1&keyword=NNILzh'%2b(select*from(select(sleep(20)))a)%2b'
```

Decoded vulnerable value:

```text
NNILzh'+(select*from(select(sleep(20)))a)+'
```

## Verification

To validate the finding responsibly in an authorized local environment:

1. Submit a normal search request and record the response time.
2. Submit the supplied test request and record the response time.
3. Repeat both requests several times to reduce the effect of network or server variability.
4. Confirm that the test request consistently takes approximately 20 seconds longer than the control request.
5. Repeat the test with a shorter delay and confirm that the observed response time changes accordingly.

A single slow response is insufficient by itself to establish SQL injection.

## Expected Result

The `keyword` value should be processed solely as search text. SQL syntax submitted through the parameter should not affect query execution or response time.

## Observed Result

When the supplied SQL expression is processed, the application allegedly delays its response by approximately 20 seconds, indicating that the database evaluated the injected `SLEEP()` expression.

## Impact

Successful exploitation could allow an unauthenticated attacker to infer sensitive database information through response-time measurements.

Potentially exposed information may include:

* User and administrator records
* Authentication data and password hashes
* Customer contact information
* Product and inventory data
* Order and transaction records
* Database names, table names and column names
* Application configuration data stored in the database

The vulnerability may also cause resource exhaustion if numerous requests invoke long database delays. The practical impact depends on database permissions, application exposure, query structure and server configuration.

## Attack Preconditions

Exploitation requires:

* Network access to the affected endpoint
* The ability to submit the `keyword` parameter
* A database query that incorporates the parameter insecurely
* Sufficiently stable response timing to distinguish delayed and non-delayed responses

No victim interaction is required.

## Remediation

### Use parameterized queries

The application should use prepared statements for every database query containing user-controlled values.

Example using MySQLi:

```php
<?php

$keyword = $_POST['keyword'] ?? '';

if (!is_string($keyword) || mb_strlen($keyword) > 100) {
    http_response_code(400);
    exit('Invalid search keyword.');
}

$searchTerm = '%' . $keyword . '%';

$stmt = $connection->prepare(
    'SELECT product_id, product_title, product_price
     FROM products
     WHERE product_title LIKE ?'
);

if ($stmt === false) {
    http_response_code(500);
    exit('Unable to prepare search query.');
}

$stmt->bind_param('s', $searchTerm);
$stmt->execute();

$result = $stmt->get_result();
```

### Apply server-side validation

Restrict the search value to a reasonable length and expected character set. Validation can reduce malformed input, but it must not replace prepared statements.

### Use a restricted database account

The application's database account should have only the permissions required for normal operation. It should not have administrative privileges or unrestricted access to unrelated databases.

### Avoid exposing database errors

Detailed SQL errors should be logged securely on the server rather than returned to users.

### Add defensive monitoring

Log abnormal search requests, repeated timing patterns and malformed input. Rate limiting may reduce automated exploitation and resource exhaustion, but it does not correct the underlying SQL injection vulnerability.

## Secure Coding Guidance

Do not attempt to fix the issue using only:

* Manual quote escaping
* Removal of selected SQL keywords
* Client-side validation
* Regular-expression blocklists
* Web application firewall rules

These controls may be bypassed. Parameterized queries are the primary remediation.

## Conclusion

The Online Shopping System in PHP search endpoint appears to permit user-controlled SQL expressions through the `keyword` parameter. The supplied request uses a time-delay expression that, when accompanied by a consistent 20-second response delay, demonstrates time-based blind SQL injection.

The application should replace dynamically constructed SQL statements with prepared queries, validate search input, restrict database privileges and review all other endpoints for similar query-construction patterns.



<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/9c6858b3-629d-432f-8f22-13ffb3c3df74" />
