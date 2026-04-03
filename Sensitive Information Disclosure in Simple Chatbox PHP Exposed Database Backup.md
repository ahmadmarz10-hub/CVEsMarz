## Sensitive Information Disclosure in Simple Chatbox PHP Exposed Database Backup
## Credit

Discovered by: Ahmad Marzook

## Product

Simple Chatbox in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/simple-chatbox-in-php-with-source-code/

## Affected Version

Simple Chatbox in PHP v1.0

## Vulnerability Type

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Category

A05:2021 – Security Misconfiguration

## Severity

High

## Description

The Simple Chatbox in PHP v1.0 is vulnerable to Sensitive Information Disclosure due to an exposed SQL database backup file.

The application includes a database dump file (chatbox.sql) located within a publicly accessible directory inside the web root. Because the web server does not restrict access to .sql files, any unauthenticated user can directly access and download the database dump via HTTP.

The exposed file is accessible at:

http://localhost/SimpleChatbox_PHP/chatbox/database/chatbox.sql

The SQL dump file contains the complete database schema and stored application data, including chat messages, user-related data, and system structure. Since the file is stored in a web-accessible location without access controls, attackers can retrieve sensitive information without authentication.

## Vulnerability Overview
Exposed SQL Database File

A database dump file is located in a publicly accessible directory:

/SimpleChatbox_PHP/chatbox/database/chatbox.sql

Because the web server allows direct access, the file can be downloaded by any remote user.

## Root Cause

The vulnerability exists due to improper server configuration and insecure handling of backup files.

Specifically:

Database backup files are stored inside the web root directory

The web server allows direct access to .sql files

No authentication or authorization controls are enforced

Sensitive files are exposed through publicly accessible paths

This leads to unintended exposure of sensitive data.

## Affected Resource
/SimpleChatbox_PHP/chatbox/database/chatbox.sql
## Proof of Concept
Steps to Reproduce

Install the Simple Chatbox in PHP.

Open a web browser.

Navigate to:

http://localhost/SimpleChatbox_PHP/chatbox/database/chatbox.sql

Observe that the SQL file is directly accessible and downloadable.

Example Extracted Data
-- Example snippet from exposed database

INSERT INTO `messages` (`id`, `username`, `message`) VALUES
(1, 'user1', 'Hello world');

INSERT INTO `users` (`id`, `username`, `password`) VALUES
(1, 'admin', 'admin123');
Impact

Successful exploitation allows attackers to access sensitive information such as:

Chat messages

Usernames and credentials

Application structure and schema

Internal data relationships

This may lead to:

Privacy violations

Credential theft

Account compromise

Further attacks on the application

Full database exposure

Vulnerability Classification
CWE

CWE-200 – Exposure of Sensitive Information

OWASP Top 10

A05:2021 – Security Misconfiguration

Suggested Remediation
Remove SQL Files from Web Root

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

Limit access to authorized users only

Avoid exposing backup files via HTTP

Additional Security Measures

Disable directory listing

Apply strict file permissions

Perform regular security audits

## PoC

<img width="1455" height="839" alt="Image" src="https://github.com/user-attachments/assets/80159b7f-47e3-42a6-b00e-ad045875ef3c" />
