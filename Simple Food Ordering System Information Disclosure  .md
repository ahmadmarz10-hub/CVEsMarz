Vulnerability Report CVE

Product: Simple Food Ordering System in PHP
Vendor: Code-Projects
Affected Version: 1.0
Vendor URL: https://code-projects.org/simple-food-order-system-in-php-with-source-code/

Vulnerability Type: Sensitive Information Disclosure / Exposed Database Backup
CWE: CWE-200 (Exposure of Sensitive Information)
OWASP Category: Security Misconfiguration
Severity: High

1. Vulnerability Overview

The Simple Food Ordering System in PHP 1.0 exposes a database backup file (food.sql) within a publicly accessible directory.

The file can be accessed directly via the web server:

http://localhost/food/sql/food.sql

Because the SQL file is stored inside the web root directory and lacks proper access restrictions, any remote user can download it without authentication.

This results in sensitive database information disclosure, including the full database schema and stored data.

2. Affected Product

Product: Simple Food Ordering System
Vendor: Code-Projects
Affected Version: 1.0

The application is a PHP-based food ordering system that includes an admin panel for managing orders, products, and categories.

3. Vulnerability Details
Exposed SQL Database Backup

A database dump file (food.sql) is stored in the following directory:

/food/sql/

Since the directory is accessible through the web server, attackers can directly retrieve the database dump.

Vulnerable Resource
http://localhost/food/sql/food.sql
Root Cause

Backup files stored in the web root

No access restrictions for .sql files

Improper server configuration

4. Proof of Concept

Deploy the application.

Open a browser.

Navigate to the following URL:

http://localhost/food/sql/food.sql

The database dump downloads or is displayed in the browser.

Example snippet from the exposed file:

-- Dumping data for table `users`
--

INSERT INTO `users` (`id`, `role`, `name`, `username`, `password`, `email`, `address`, `contact`, `verified`, `deleted`) VALUES
(1, 'Administrator', 'Admin 1', 'root', 'toor', '', 'Address 1', 9898000000, 1, 0),
(2, 'Customer', 'Customer 1', 'user1', 'pass1', 'mail2@example.com', 'Address 2', 9898000001, 1, 0),
(3, 'Customer', 'Customer 2', 'user2', 'pass2', 'mail3@example.com', 'Address 3', 9898000002, 1, 0),
(4, 'Customer', 'Customer 3', 'user3', 'pass3', '', '', 9898000003, 0, 0),
(5, 'Customer', 'Customer 4', 'user4', 'pass4', '', '', 9898000004, 0, 1);

An attacker can extract database contents including administrative credentials.

5. Impact

Successful exploitation allows attackers to obtain sensitive information such as:

Administrator credentials

User information

Order records

Product data

Database schema

This information may lead to:

Account compromise

Credential cracking

Database manipulation

Further attacks against the application

In severe cases, attackers may gain administrative access to the system.

6. Remediation

The following security measures should be implemented:

1. Remove SQL Files from Web Root

Store database backups outside the public web directory.

Example secure location:

/var/backups/
2. Restrict Access to SQL Files

Apache configuration:

<Files "*.sql">
  Require all denied
</Files>

Nginx configuration:

location ~* \.sql$ {
    deny all;
}
3. Use Secure Backup Storage

Database backups should be stored in:

secured storage

protected directories

restricted access environments

7. Vulnerability Classification

CWE:

CWE-200 – Exposure of Sensitive Information

OWASP Top 10:

A05:2021 – Security Misconfiguration

8. References

Code-Projects – Simple Food Ordering System
https://code-projects.org/simple-food-order-system-in-php-with-source-code/

CWE-200: Exposure of Sensitive Information
https://cwe.mitre.org/data/definitions/200.html

OWASP Security Misconfiguration
https://owasp.org/Top10/

<img width="1471" height="854" alt="Image" src="https://github.com/user-attachments/assets/bcafbaa5-c38d-4691-bdde-75ea054d3927" />
