## Sensitive Information Disclosure in Simple Inventory System PHP Exposed Database Backup
## Credit

Discovered by: Ahmad Marzook

## Product

Simple Inventory System in PHP

## Vendor

Code-Projects

## Vendor Homepage

https://code-projects.org/simple-inventory-system-in-php-with-source-code/

## Affected Version

Simple Inventory System in PHP v1.0

## Vulnerability Type

Sensitive Information Disclosure / Exposed Database Backup

CWE

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

OWASP Top 10

A05:2021 – Security Misconfiguration

## Severity

High

CVSS v3.1 Score

7.5 (High)

Vector

CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
Description

The Simple Inventory System in PHP v1.0 is vulnerable to Sensitive Information Disclosure due to an exposed SQL database backup file.

The application stores a complete MySQL database dump (inventorymanagement.sql) inside a directory that is publicly accessible through the web server. Because the backup file resides within the web root and no access restrictions are enforced, any remote user can directly access and download the database without authentication.

## The exposed resource is located at:

/InventoryManagement/Database_Backup/inventorymanagement.sql

The database dump contains the complete database schema together with all stored application data. Depending on the deployment, this information may include administrator accounts, employee information, supplier records, customer information, inventory data, transaction history, authentication credentials, and other confidential business information.

Since the backup file is accessible through a direct HTTP request, an attacker does not need valid credentials or any interaction with the application to obtain sensitive information. The vulnerability results from insecure deployment practices where backup files are stored in publicly accessible directories instead of protected locations outside the web root.

Successful exploitation allows attackers to retrieve the entire backend database, facilitating further attacks such as credential theft, password cracking, account compromise, database enumeration, information gathering, and additional attacks against the application infrastructure.

## Root Cause

The vulnerability exists because sensitive database backup files are deployed inside directories that are directly accessible through the web server.

Specifically:

Database backup files are stored inside the public web root.
The web server permits direct access to .sql files.
No authentication is required before downloading backup files.
No authorization or access control protects sensitive resources.
Backup files are exposed through predictable URLs.

This insecure configuration allows any remote user to obtain confidential application data.

## Affected Resource

/InventoryManagement/Database_Backup/inventorymanagement.sql
## HTTP Method
GET
## Proof of Concept
Full HTTP Request
 ```plain
GET /InventoryManagement/Database_Backup/inventorymanagement.sql HTTP/1.1
Host: localhost
sec-ch-ua: "Not-A.Brand";v="24", "Chromium";v="146"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=8sgapf310jnc0psdjc89q68h8e
Connection: keep-alive
 ```

## Steps to Reproduce
Install the Simple Inventory System in PHP.
Start the web server.
Open a web browser.
Navigate to:
http://localhost/InventoryManagement/Database_Backup/inventorymanagement.sql
Observe that the SQL database backup is downloaded or displayed without authentication.
Result

The complete database backup is disclosed to unauthenticated users through a direct HTTP request.

## Impact

An attacker can exploit this vulnerability to obtain sensitive information including:

Administrator accounts
User credentials
Password hashes or plaintext passwords (depending on implementation)
Customer records
Supplier information
Inventory records
Product information
Sales and transaction history
Internal database structure
Business-sensitive information

Disclosure of this information may result in:

Unauthorized access to administrator accounts
Credential reuse attacks
Offline password cracking
Data theft
Business intelligence leakage
Facilitation of additional attacks such as SQL injection, privilege escalation, or account takeover
Vulnerability Classification
CWE
CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor
OWASP Top 10
A05:2021 – Security Misconfiguration
Suggested Remediation
Remove Backup Files from the Web Root

Database backup files should never be stored inside publicly accessible directories.

For example:

/var/backups/inventory/
Restrict Access to SQL Files
Apache
<FilesMatch "\.(sql|bak|backup)$">
    Require all denied
</FilesMatch>
Nginx
location ~* \.(sql|bak|backup)$ {
    deny all;
    return 403;
}
Store Backups Securely
## Store backups outside the document root.
Restrict filesystem permissions.
Limit access to authorized administrators.
Encrypt backup files where appropriate.


<img width="1480" height="843" alt="Image" src="https://github.com/user-attachments/assets/c9ff4cc4-84b2-419a-ac97-677a4a815894" />
