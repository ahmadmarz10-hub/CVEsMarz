## Sensitive Information Disclosure in Online FIR System PHP Exposed Database Backup
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

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

## Severity

High

## Description

The Online FIR System in PHP v1.0 is vulnerable to Sensitive Information Disclosure due to an exposed SQL database backup file.

The application stores a database dump file (complaints.sql) inside a publicly accessible directory within the web root. Because the web server does not restrict access to .sql files, any unauthenticated user can directly access and download the database dump via HTTP.

The exposed file is accessible at:

http://localhost/Online_FIR_System/complaints.sql

Since SQL dump files contain the full database schema and stored application data, unauthorized users can retrieve sensitive information such as user accounts, complaint records, and administrative data.

Applications of this type typically manage complaint and user data through a MySQL backend, meaning exposed SQL files can reveal complete datasets including credentials and operational data .

This issue arises from improper server configuration and insecure storage of backup files in web-accessible locations.

## Vulnerability Overview
Exposed SQL Database File

A database dump file is located in a publicly accessible directory:

/Online_FIR_System/complaints.sql

Because the file resides inside the web root and lacks access restrictions, it can be downloaded directly by any remote user.

## Root Cause

The vulnerability is caused by insecure deployment practices and server misconfiguration.

Specifically:

Database backup files are stored inside the web root directory

The web server allows direct access to .sql files

No authentication or authorization is required to access backup files

No server-side restrictions are configured to block access to sensitive file types

As a result, sensitive database contents are exposed to unauthorized users.

## Affected Resource
/Online_FIR_System/complaints.sql
## Proof of Concept
Steps to Reproduce

Install and run the Online FIR System in PHP.

Open a web browser.

Navigate to:

http://localhost/Online_FIR_System/complaints.sql

Observe that the SQL file is directly accessible and downloadable.

## Extracted Data
 snippet from exposed database dump

--
-- Dumping data for table `cops`
--

INSERT INTO `cops` (`id`, `Name`, `Post`, `Category`, `Email`, `Password`, `Count`) VALUES
(1, 'Pradhyuman', 'SI', 'Murder', 'pradh@gmail.com', 'daya', 0),
(3, 'Abhijeet', 'ASI', '', 'abhijeet@gmail.com', 'abhi', 0),
(4, 'Mayank Wadhwani', 'SI', 'Theft & Robbery', 'mayank@gmail.com', 'manku', 0),
(6, 'Anubhav Tyagi', 'SI', 'Domestic Affairs', 'tyagi@gmail.com', 'tyagi', 0),
(7, 'Manan Gupta', 'SI', 'Others', 'gupta@gmail.com', 'man', 0),
(8, 'Piyush Gupta', 'SI', 'Theft & Robbery', 'piyu@gmail.com', 'piyu', 0),
(9, 'Ravi Shankar', 'SI', 'Domestic Affairs', 'shank@gmail.com', 'ravi', 1),
(10, 'Anant Sachan', 'SI', 'Murder', 'sachan@gmail.com', 'anant', 0),
(11, 'Shubham Maurya', 'SI', 'Others', 'maurya@gmail.com', 'maurya', 0),
(12, 'Abhishek Jaiswal', 'ASI', '', 'jaiswal@gmail.com', 'abhi', 0),
(13, 'Aman Raj', 'ASI', '', 'raj@gmail.com', 'aman', 0);

## Impact

Successful exploitation of this vulnerability allows attackers to access sensitive information such as:

Administrator credentials

User account details

Complaint records

Personal user information

Database schema

This may lead to:

Account compromise

Credential reuse attacks

Unauthorized administrative access

Data manipulation or deletion

Further exploitation of the system

In severe cases, attackers may gain full control over the application database.

## Vulnerability Classification
CWE

CWE-200 – Exposure of Sensitive Information

OWASP Top 10

A05:2021 – Security Misconfiguration

## Suggested Remediation
Remove SQL Files from Web Root

Database backups should not be stored in publicly accessible directories.

Example secure location:

/var/backups/
Restrict Access to SQL Files
Apache Configuration
<Files "*.sql">
    Require all denied
</Files>
Nginx Configuration
location ~* \.sql$ {
    deny all;
}
Secure Backup Storage

Store database backups in:

Restricted directories

Internal storage systems

Non-public environments

Access should be limited to authorized administrators only.

Additional Security Measures

Disable directory listing

Apply strict file permissions

Regularly audit exposed resources

## PoC 

<img width="1459" height="828" alt="Image" src="https://github.com/user-attachments/assets/576cb467-9fde-4faa-8bff-00664fe20a95" />
