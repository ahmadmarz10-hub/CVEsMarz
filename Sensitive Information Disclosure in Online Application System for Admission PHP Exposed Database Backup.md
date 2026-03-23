## Sensitive Information Disclosure in Online Application System for Admission PHP Exposed Database Backup
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

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

## Severity

High

## Description

The Online Application System for Admission in PHP v1.0 is affected by a Sensitive Information Disclosure vulnerability due to an exposed SQL database backup file.

The application stores a database dump file (oas.sql) inside a publicly accessible directory within the web root. Because the web server does not restrict access to .sql files, any remote user can directly access and download the database dump without authentication.

The exposed file can be accessed via:

http://localhost/OnlineApplicationSystem_PHP/enrollment/database/oas.sql

Since the SQL file contains the complete database structure and stored application data, an attacker can retrieve sensitive information including user records, credentials, application data, and database schema.

This vulnerability arises from improper server configuration and insecure storage of backup files inside web-accessible directories.

## Vulnerability Overview
Exposed Database Backup

A database dump file is stored in a publicly accessible location:

/enrollment/database/oas.sql

Because this directory is accessible via HTTP, the file can be downloaded directly without authentication.

## Root Cause

The vulnerability exists due to insecure server configuration and improper handling of backup files.

Specifically:

Database backup files are stored inside the web root directory

The web server allows direct access to .sql files

No authentication or authorization is required to access backup files

No access control rules are implemented to restrict sensitive file exposure

As a result, sensitive database content is exposed to unauthorized users.

## Affected Resource
/OnlineApplicationSystem_PHP/enrollment/database/oas.sql
## Proof of Concept
## Steps to Reproduce

Install and run the Online Application System for Admission in PHP.

Open a web browser.

Navigate to the following URL:

http://localhost/OnlineApplicationSystem_PHP/enrollment/database/oas.sql

Observe that the SQL file is directly accessible and downloadable.

 ## Extracted Data
 content from exposed database dump

-- Dumping data for table `t_admin`


INSERT INTO `t_admin` (`ad_id`, `ad_name`, `ad_pswd`, `ad_eml`) VALUES
('AD00000001', 'admin', 'admin', 'admin@gmail.com'),
('AD00002', 'Dilraj', 'QCoxFrwx', 'dilrajkaur18@gmail.com');

 ## Impact

Successful exploitation of this vulnerability allows attackers to access sensitive information including:

Administrator credentials

User account data

Password hashes or plaintext passwords

Admission records

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

Database backups should never be stored in publicly accessible directories.

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

Store backup files in:

Restricted directories

Internal storage systems

Locations not accessible via HTTP

Access should be limited to authorized administrators only.

Security Hardening

Disable directory listing

Implement proper file permissions

Regularly audit exposed files

 ## PoC 

<img width="1340" height="838" alt="Image" src="https://github.com/user-attachments/assets/f9d6f3ba-a7ba-413d-baac-99a57e570713" />
