## Online Food Ordering System in PHP 1.0 – Sensitive Information Disclosure

## Credit

Discovered by:
Ahmad Marzook

## Product

Online Food Ordering System in PHP

## Vendor

Code-Projects

## Affected Version

1.0

## Vendor URL

https://code-projects.org/online-food-ordering-system-in-php-with-source-code/

## Vulnerability Type

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

## Severity

High

## 1. Vulnerability Overview

The Online Food Ordering System in PHP 1.0 exposes a database backup file (localhost.sql) within a publicly accessible directory. Because the SQL file is stored inside the web root and lacks proper access restrictions, any remote user can directly access and download the database dump without authentication.

## The file can be accessed through the web server at the following location:

http://localhost/dbfood/localhost.sql

Since the database dump contains the full database schema and stored application data, unauthorized users can retrieve sensitive information including user credentials, administrative accounts, and application data.

This issue occurs due to improper server configuration and insecure storage of database backups in publicly accessible directories.

## 2. Affected Product


The application is a PHP-based food ordering system that allows users to browse food items, place orders, and manage food listings through an administrative panel.

## 3. Vulnerability Details
Exposed SQL Database Backup

A database dump file (localhost.sql) is stored inside a web-accessible directory.

##  location:

/dbfood/localhost.sql

Because the directory is publicly accessible through the web server, an attacker can retrieve the database dump directly without authentication.

## Vulnerable Resource
http://localhost/dbfood/localhost.sql

## Root Cause

The vulnerability exists due to improper handling of backup files and insecure server configuration.

Specifically:

Database backup files are stored inside the public web root directory

The web server allows direct access to .sql files

No authentication or access restrictions protect backup files

Sensitive database information is exposed through publicly accessible resources

Because of this misconfiguration, attackers can directly download the database dump.

## 4. Proof of Concept

Deploy the Online Food Ordering System in PHP application.

Open a web browser.

Navigate to the following URL:

http://localhost/dbfood/localhost.sql

## The database dump file is downloaded or displayed in the browser.

Example snippet from the exposed file:
 ```plain
-- Dumping data for table `tbadmin`
--

INSERT INTO `tbadmin` (`fld_id`, `fld_username`, `fld_password`) VALUES
(1, 'admin', 'admin@123'); 
 ```

An attacker can extract sensitive database contents including administrative credentials and user information.

## 5. Impact

Successful exploitation allows attackers to obtain sensitive information such as:

Administrator credentials

User account information

Password hashes or plaintext passwords

Order records

Food product data

Database schema

This information may lead to:

Account compromise

Credential cracking

Database manipulation

Unauthorized administrative access

Further attacks against the application

In severe cases, attackers may gain full control over the application database.

## 6. Remediation

Developers should implement the following security measures.

Remove SQL Files from Web Root

Database backups must not be stored in publicly accessible directories.

## Example secure location:

/var/backups/
Restrict Access to SQL Files
Apache configuration
<Files "*.sql">
Require all denied
</Files>
Nginx configuration
location ~* \.sql$ {
deny all;
}
Use Secure Backup Storage

Database backups should be stored in:

secure backup storage

restricted directories

internal systems not accessible via HTTP

Access to backup files should be limited to administrators only.

## 7. Vulnerability Classification
CWE

CWE-200 – Exposure of Sensitive Information

OWASP Top 10

A05:2021 – Security Misconfiguration

## 8. References

Online Food Ordering System Project
https://code-projects.org/online-food-ordering-system-in-php-with-source-code/

CWE-200: Exposure of Sensitive Information
https://cwe.mitre.org/data/definitions/200.html

OWASP Top 10 – Security Misconfiguration
https://owasp.org/Top10/

PoC 

<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/4d12c63a-d010-4e65-95ca-392f2d095e32" />
