## Sensitive Information Disclosure in Patient Record Management System PHP Exposed Database Backup
## Credit

Discovered by: Ahmad Marzook

## Product

Patient Record Management System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/patient-record-management-system-in-php-with-source-code/

## Affected Version

Patient Record Management System v1.0

## Vulnerability Type

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

## Severity

High

## Description

The Patient Record Management System in PHP v1.0 is vulnerable to Sensitive Information Disclosure due to an exposed SQL database backup file.

The application stores a database dump file (hcpms.sql) inside a publicly accessible directory within the web root. Because the web server does not restrict access to .sql files, any remote attacker can directly access and download the database dump without authentication.

The exposed file can be accessed at:

http://localhost/HCPMS%20PHP/Health%20Care%20Patient%20Record%20Management%20System/db/hcpms.sql

The SQL dump contains the complete database structure and application data. Since PHP applications often store sensitive user and system data in databases, exposing such files may lead to severe data leakage risks.

This vulnerability allows unauthorized users to retrieve sensitive information such as patient records, administrative credentials, and system data.

## Vulnerability Overview
Exposed SQL Database Backup

A database dump file is stored in a publicly accessible directory:

/HCPMS PHP/Health Care Patient Record Management System/db/hcpms.sql

Because the directory is accessible via HTTP, the database file can be downloaded directly without authentication.

## Root Cause

The vulnerability exists due to improper server configuration and insecure handling of backup files.

Specifically:

Database backup files are stored inside the web root directory

The web server allows direct access to .sql files

No authentication or authorization controls are applied

No restrictions exist to prevent direct file access

As a result, sensitive database content is exposed to unauthorized users.

## Affected Resource
/HCPMS PHP/Health Care Patient Record Management System/db/hcpms.sql
## Proof of Concept
## Steps to Reproduce

Install the Patient Record Management System in PHP.

Open a web browser.

Navigate to the following URL:

http://localhost/HCPMS%20PHP/Health%20Care%20Patient%20Record%20Management%20System/db/hcpms.sql

Observe that the SQL file is directly accessible and downloadable.

## Extracted Data
 snippet from exposed database

-- Dumping data for table `admin`
--

INSERT INTO `admin` (`admin_id`, `username`, `password`, `firstname`, `middlename`, `lastname`) VALUES
(2, 'admin', 'admin', 'Administrator', '', '');

-- --------------------------------------------------------
## Impact

Successful exploitation allows attackers to access sensitive data such as:

Patient medical records

Administrator credentials

User account information

Personal identifiable information (PII)

Database schema

This may lead to:

Unauthorized access to patient data

Privacy violations

Credential theft

Account takeover

Full database compromise

Further targeted attacks

PHP applications are particularly vulnerable to data leakage when sensitive files are improperly exposed.

## Vulnerability Classification
CWE

CWE-200 – Exposure of Sensitive Information

OWASP Top 10

A05:2021 – Security Misconfiguration

## Suggested Remediation
Remove Backup Files from Web Root

Store database backups outside publicly accessible directories.

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

Use restricted directories

Store backups in internal systems

Limit access to authorized administrators only

Additional Security Measures

Disable directory listing

Apply strict file permissions

Regularly audit exposed files

## PoC 

<img width="1477" height="848" alt="Image" src="https://github.com/user-attachments/assets/b18d8b15-76ba-4962-a766-51d754a0a350" />
