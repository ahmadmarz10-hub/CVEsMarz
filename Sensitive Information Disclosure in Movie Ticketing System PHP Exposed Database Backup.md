## Sensitive Information Disclosure in Movie Ticketing System PHP Exposed Database Backup
##  Credit

Discovered by: Ahmad Marzook

##  Product

Movie Ticketing System in PHP

##  Vendor

Code-Projects

##  Vendor Homepage

https://code-projects.org/movie-ticketing-system-in-php-with-source-code/

##  Affected Version

Movie Ticketing System v1.0

##  Vulnerability Type

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

##  Severity

High

##  Description

The Movie Ticketing System in PHP v1.0 is vulnerable to Sensitive Information Disclosure due to an exposed SQL database backup file.

The application stores a database dump file (moviedb.sql) inside a publicly accessible directory within the web root. Because the web server does not restrict access to .sql files, any remote attacker can directly access and download the database dump without authentication.

The exposed file can be accessed at:

http://localhost/movie/db/moviedb.sql

The SQL dump file contains the full database structure and stored application data. Since this application is built using PHP and MySQL, it stores sensitive operational data such as user accounts, booking information, and administrative credentials in the database.

Because the file is publicly accessible, an attacker can retrieve sensitive information directly through the browser without any authentication.

##  Vulnerability Overview
Exposed SQL Database Backup

A database dump file is located in a publicly accessible directory:

/movie/db/moviedb.sql

The web server allows direct access to this file, enabling unauthorized users to download the database contents.

##  Root Cause

The vulnerability exists due to insecure server configuration and improper handling of backup files.

Specifically:

Database backup files are stored inside the web root directory

The web server allows direct access to .sql files

No authentication or authorization is required

No access control rules are implemented to restrict sensitive file access

As a result, sensitive database contents are exposed to unauthorized users.

##  Affected Resource
/movie/db/moviedb.sql
##  Proof of Concept
Steps to Reproduce

Install the Movie Ticketing System in PHP.

Open a web browser.

Navigate to the following URL:

http://localhost/movie/db/moviedb.sql

Observe that the SQL file is directly accessible and downloadable.

 Extracted Data
-- Example snippet from exposed database dump

--
-- Dumping data for table `user`
--

INSERT INTO `user` (`userId`, `userName`, `password`, `status`) VALUES
(1, 'admin', 'admin', 101),
(3, 'user', 'user', 202);

## Impact

Successful exploitation allows attackers to access sensitive information such as:

Administrator credentials

User account data

Booking and transaction records

Email addresses and personal information

Database schema

This may lead to:

Account compromise

Credential reuse attacks

Unauthorized administrative access

Data manipulation or deletion

Further exploitation of the application

Multiple vulnerabilities have already been reported in similar movie ticket systems, highlighting the risk of poor input validation and data exposure in such applications.

##  Vulnerability Classification
CWE

CWE-200 – Exposure of Sensitive Information

OWASP Top 10

A05:2021 – Security Misconfiguration

##  Suggested Remediation
Remove Backup Files from Web Root

Database backups should not be stored in publicly accessible directories.

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

<img width="1370" height="843" alt="Image" src="https://github.com/user-attachments/assets/670d4c52-177d-4f2d-9b14-1a1d04087d74" />
