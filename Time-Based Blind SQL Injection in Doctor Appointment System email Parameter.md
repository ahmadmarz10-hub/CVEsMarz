## Time-Based Blind SQL Injection in Doctor Appointment System email Parameter
## Credit

Discovered by: Ahmad Marzook

## Product

Doctor Appointment System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/doctor-appointment-system-in-php-with-source-code/

## Affected Version

Doctor Appointment System in PHP v1.0

## Vulnerability Type

Time-Based Blind SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements Used in an SQL Command ('SQL Injection')

OWASP Top 10

A03:2021 – Injection

## Severity

High

CVSS v3.1 Score

8.8 (High)

Vector

CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:L
Description

The Doctor Appointment System in PHP v1.0 is vulnerable to a Time-Based Blind SQL Injection vulnerability in the patient authentication functionality.

The vulnerability exists in the patient_login.php endpoint, where user-supplied input provided through the email parameter is incorporated into a backend SQL query without using parameterized statements or sufficient input validation.

An attacker can inject specially crafted SQL expressions into the email parameter to manipulate the SQL query executed by the database server. By using time-delay functions such as SLEEP(), an attacker can determine whether injected SQL statements are executed based on measurable response delays, even when the application does not display SQL errors or query results.

For example, supplying a payload containing the MySQL SLEEP(20) function causes the application to delay its response by approximately 20 seconds if the injected SQL expression is successfully evaluated. This confirms that the application is vulnerable to blind SQL injection.

Because the vulnerability occurs during the authentication process, an attacker can remotely interact with the affected endpoint without requiring valid credentials. Once confirmed, the vulnerability may be leveraged to enumerate the database structure, extract sensitive information, recover administrator and patient records, obtain password hashes, bypass authentication under certain conditions, and facilitate further attacks against the application.

The issue is caused by unsafe construction of SQL queries using untrusted user input and the absence of prepared statements. Successful exploitation may result in unauthorized disclosure of confidential information and compromise of the application's database.

## Root Cause

The application directly incorporates user-controlled input from the email parameter into an SQL query without using prepared statements or parameterized queries.

A vulnerable implementation resembles:

$email = $_POST['email'];
$password = $_POST['password'];

$sql = "SELECT * FROM patient
        WHERE email='$email'
        AND password='$password'";

$result = mysqli_query($conn, $sql);

Because the SQL statement is dynamically constructed through string concatenation, specially crafted input supplied by an attacker can alter the intended query structure.

## Affected Endpoint
POST /das/patient_login.php
## Vulnerable Parameter
email
## HTTP Method
POST
## Proof of Concept
Full HTTP Request
 ```plain
POST /das/patient_login.php HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="146", "Not=A?Brand";v="8", "Chromium";v="146"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=b3sb4v8j13f9q5kbbokqi1qmdc
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/das/patient_login.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 99

email=adnim@burpcollaborator.net'%2b(select*from(select(sleep(20)))a)%2b'&password=13020944&submit=
 ```
## Steps to Reproduce
Install the Doctor Appointment System in PHP.
Navigate to the patient login page.
Intercept the login request using Burp Suite.
Replace the value of the email parameter with a time-delay SQL payload.
Submit the request.
Observe that the server response is delayed by approximately 20 seconds.
Repeat the request multiple times to verify that the delay is consistent and reproducible.
Result

The application delays its response when processing the injected SQL expression, indicating that user-supplied input is executed as part of the backend SQL query and confirming the presence of a Time-Based Blind SQL Injection vulnerability.

## Impact

Successful exploitation may allow an attacker to:

Enumerate database names, tables, and columns.
Extract sensitive patient and administrator information.
Retrieve password hashes.
Bypass authentication in vulnerable query implementations.
Access confidential medical records.
Enumerate application configuration data.
Facilitate additional attacks against the application.

Because the vulnerability is remotely exploitable and requires no authentication, it poses a significant risk to the confidentiality and integrity of the application.

Vulnerability Classification
CWE
CWE-89 – SQL Injection
OWASP Top 10
A03:2021 – Injection
Suggested Remediation
Use Prepared Statements

Replace dynamically constructed SQL queries with parameterized queries.

Example:

$stmt = $conn->prepare(
    "SELECT * FROM patient WHERE email = ? AND password = ?"
);

$stmt->bind_param("ss", $email, $password);
$stmt->execute();
Validate Input

Validate email addresses using server-side validation (e.g., filter_var($email, FILTER_VALIDATE_EMAIL)) before processing them.

Least-Privilege Database Accounts

Ensure that the database account used by the application has only the minimum privileges required for normal operation.

Error Handling

Disable verbose SQL error messages in production environments and log detailed errors securely on the server.

Security Testing

Perform regular secure code reviews and penetration testing to identify and remediate SQL injection vulnerabilities throughout the application.


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/be2df20b-43d6-473e-affa-33825f9294aa" />
