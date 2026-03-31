## Sensitive Information Disclosure in Online Library Management System PHP Exposed Database Backup
## Credit

Discovered by: Ahmad Marzouk

## Product

Online Library Management System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/online-library-management-system-in-php-with-source-code/

## Affected Version

Online Library Management System v1.0

## Vulnerability Type

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

## Severity

High

## Description

The Online Library Management System in PHP v1.0 is vulnerable to Sensitive Information Disclosure due to an exposed SQL database backup file.

The application includes a database dump file (library.sql) within a publicly accessible directory under the web root. Because the web server does not restrict access to .sql files, any unauthenticated user can directly access and download the database dump via HTTP.

The exposed file can be accessed at:

http://localhost/Library/sql/library.sql

The database dump contains the full database schema and stored application data. This type of system typically manages sensitive information such as user accounts, student records, issued books, and administrative credentials.

Because the file is stored inside a web-accessible directory and lacks access control, attackers can retrieve sensitive data without authentication.

## Vulnerability Overview
Exposed SQL Database File

A database dump file is located in a publicly accessible directory:

/Library/sql/library.sql

The web server allows direct access to this file, enabling unauthorized users to download the database contents.

## Root Cause

The vulnerability exists due to insecure deployment practices and improper server configuration.

Specifically:

Database backup files are stored inside the web root directory

The web server allows direct access to .sql files

No authentication or authorization is required

No access control rules are implemented to restrict sensitive files

As a result, sensitive database contents are exposed to unauthorized users.

## Affected Resource
/Library/sql/library.sql
## Proof of Concept
Steps to Reproduce

Install the Online Library Management System in PHP.

Open a web browser.

Navigate to the following URL:

http://localhost/Library/sql/library.sql

Observe that the SQL file is directly accessible and downloadable.

 Extracted Data
-- Example snippet from exposed database
--
-- Dumping data for table `admin`
--

INSERT INTO `admin` (`id`, `FullName`, `AdminEmail`, `UserName`, `Password`, `updationDate`) VALUES
(1, 'Anuj Kumar', 'phpgurukulofficial@gmail.com', 'admin', 'f925916e2754e5e03f75dd58a5733251', '2017-07-16 18:11:42');

## Impact

Successful exploitation allows attackers to access sensitive information such as:

Administrator credentials

Student/user account data

Issued book records

Email addresses and personal information

Database schema

This may lead to:

Account compromise

Unauthorized administrative access

Credential reuse attacks

Data manipulation or deletion

Further compromise of the application

In severe cases, attackers may gain full control over the system database.

## Vulnerability Classification
CWE

CWE-200 – Exposure of Sensitive Information

OWASP Top 10

A05:2021 – Security Misconfiguration

Suggested Remediation
Remove SQL Files from Web Root

Database backups must not be stored in publicly accessible directories.

Example:

/var/backups/
Restrict Access to SQL Files
Apache
<Files "*.sql">
    Require all denied
</Files>
Nginx
location ~* \.sql$ {
    deny all;
}
Secure Backup Storage

Store backups in restricted directories

Limit access to authorized administrators

Avoid placing sensitive files in web-accessible paths

Additional Security Measures

Disable directory listing

Apply strict file permissions

Conduct regular security audits

## PoC

<img width="1468" height="910" alt="Image" src="https://github.com/user-attachments/assets/f9b40c9a-14ba-48da-a03e-7517ca3c2c65" />
