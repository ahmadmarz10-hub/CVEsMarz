# Vehicle Management System in PHP – Sensitive Information Disclosure via Exposed Database File

## Credit

**Discovered by:** Ahmad Marzook

## Product

Vehicle Management System in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/vehicle-management-system-in-php-with-source-code/

## Affected Version

Vehicle Management System in PHP — version to be confirmed from the affected distribution.

## Vulnerability Type

Sensitive Information Disclosure / Exposed SQL Database File

## CWE

**CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor**

## OWASP Top 10

**A05:2021 — Security Misconfiguration**

## Severity

**High**

## Suggested CVSS v3.1 Score

**7.5 (High)**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
```

This score assumes that the SQL file can be retrieved without authentication and contains sensitive information. The final score should be adjusted according to the actual contents of the exposed database.

---

# Vulnerability Overview

The Vehicle Management System in PHP is affected by a Sensitive Information Disclosure vulnerability caused by an SQL database file being stored within the application's publicly accessible web directory.

The affected resource is:

```text
/Vehicle-Management/vehicle_management.sql
```

When the application is deployed with its original directory structure under the web server document root, the `vehicle_management.sql` file may be directly accessible through an HTTP request.

An unauthenticated user may therefore be able to retrieve the database file by accessing:

```text
http://localhost/Vehicle-Management/vehicle_management.sql
```

In a remotely deployed instance, the equivalent resource would potentially be accessible through the application's public hostname.

If the web server serves `.sql` files without access restrictions, an attacker can download the file without authenticating to the Vehicle Management System.

The exposed SQL file may contain the application's database schema and data included with the distributed project. Depending on the contents of the affected deployment, disclosure may expose database table structures, account records, vehicle information, booking information, driver records, authentication-related information, or other application data.

---

# Description

The Vehicle Management System in PHP is vulnerable to Sensitive Information Disclosure due to an SQL database file being located within a web-accessible application directory.

The affected file, `vehicle_management.sql`, is distributed as part of the application and is used to initialize the MySQL database during installation. When administrators deploy the complete `Vehicle-Management` directory directly under a web server document root, the SQL file remains alongside the publicly accessible PHP application files.

The vulnerable resource can be requested directly through the following application path:

```text
/Vehicle-Management/vehicle_management.sql
```

If the web server does not explicitly prohibit access to `.sql` resources, an unauthenticated attacker can send a direct HTTP GET request for the file. The server may return the SQL dump as downloadable content or display its contents directly in the HTTP response.

No interaction with the application's normal authentication mechanism is necessary because the request targets the static SQL resource directly rather than an authenticated PHP endpoint.

Database dump files can contain highly valuable information about an application. Depending on the contents of `vehicle_management.sql` and subsequent modifications made by administrators, exposed information may include database schemas, table names, column definitions, application records, user information, administrative account information, password hashes or other authentication-related values, vehicle details, driver information, booking records, and internal relationships between database objects.

Even when an exposed SQL file contains only the application's initial database rather than current production data, disclosure of the complete database structure can provide an attacker with detailed knowledge of the application's internal implementation. This information may facilitate additional attacks by revealing table names, authentication structures, database relationships, and other implementation details.

The root cause is the insecure placement of a database initialization or backup file within a directory that can be served directly by the web server. The application distribution and deployment process should ensure that database files are either removed after installation or stored outside the document root.

Successful exploitation requires only the ability to send an HTTP request to the affected server. No authentication or user interaction is required if the SQL file is publicly accessible.

The vulnerability is classified as **CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor** and represents a security misconfiguration resulting from the exposure of sensitive database resources through the application's public web directory.

---

# Root Cause

The vulnerability results from insecure storage and deployment of the application's SQL database file.

The contributing factors include:

* The `vehicle_management.sql` file is located within the application directory.
* The complete application directory may be deployed directly under the web server document root.
* The SQL file may remain accessible after database initialization.
* The web server does not necessarily block direct HTTP requests for `.sql` files.
* No authentication or authorization mechanism protects static database resources.
* Sensitive installation files are not automatically removed after deployment.

As a result, users who know or discover the predictable file path can potentially retrieve the SQL file directly.

---

# Affected Resource

```text
/Vehicle-Management/vehicle_management.sql
```

## HTTP Method

```text
GET
```

---

# Proof of Concept

## Full HTTP Request

```http
GET /Vehicle-Management/vehicle_management.sql HTTP/1.1
Host: localhost
Accept: */*
Connection: close
```

## PoC URL

```text
http://localhost/Vehicle-Management/vehicle_management.sql
```

For an authorized test installation, requesting the resource directly demonstrates the vulnerability if the web server responds with the contents of the SQL file or offers the file for download without requiring authentication.

---

# Steps to Reproduce

1. Deploy the Vehicle Management System in PHP in an authorized local testing environment.

2. Place the application under the web server document root using the original `Vehicle-Management` directory structure.

3. Configure the database as instructed by the application installation process.

4. Open a browser or HTTP client.

5. Request the following resource directly:

```text
http://localhost/Vehicle-Management/vehicle_management.sql
```

6. Observe the HTTP response.

7. If the SQL database file is displayed or downloaded without authentication, the information disclosure vulnerability is confirmed.

---

# Expected Result

Sensitive database files should not be accessible through the public web server.

A direct request for the SQL resource should return an appropriate error response, such as:

```text
403 Forbidden
```

or:

```text
404 Not Found
```

---

# Observed Result

The SQL database file is reportedly accessible through a direct request to:

```text
/Vehicle-Management/vehicle_management.sql
```

If the server returns the file contents without requiring authentication, an unauthenticated user can retrieve the exposed database resource.

---

# Impact

Successful exploitation may disclose information contained within the SQL database file, potentially including:

* Database schema and internal structure
* Table and column names
* User account information
* Administrator account information
* Authentication-related data
* Password hashes or credentials if present
* Vehicle records
* Driver information
* Booking and transaction records
* Internal application data

An attacker could potentially use the disclosed information to perform additional attacks against the application.

Possible secondary consequences include:

* Account compromise if reusable credentials are exposed
* Offline password cracking if password hashes are disclosed
* Improved reconnaissance for SQL injection attacks
* Identification of administrative accounts
* Disclosure of sensitive operational information
* Privacy violations involving application users

The precise impact depends on the actual contents of the exposed SQL file.

---

# Authentication Requirements

If the file is directly accessible as described, exploitation requires:

```text
Privileges Required: None
User Interaction: None
Attack Vector: Network
```

The attacker does not need to authenticate to the application because the SQL file is retrieved directly from the web server.

---

# Vulnerability Classification

**CWE**

CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor

**OWASP Top 10**

A05:2021 — Security Misconfiguration

---

# Remediation

## Remove SQL Files from the Public Web Root

Database initialization files and backups should never remain in publicly accessible application directories.

Move the file to a location outside the document root, for example:

```text
/var/backups/vehicle-management/
```

## Remove Installation Files After Setup

If `vehicle_management.sql` is required only during installation, administrators should delete it from the deployed application directory after the database has been successfully initialized.

## Restrict Access to SQL Files

Apache servers can explicitly block access to SQL resources:

```apache
<FilesMatch "\.(sql|bak|backup|dump)$">
    Require all denied
</FilesMatch>
```

For Nginx:

```nginx
location ~* \.(sql|bak|backup|dump)$ {
    deny all;
    return 403;
}
```

These restrictions provide defense in depth but should not replace storing sensitive files outside the document root.

## Review Deployment Packages

Production deployment processes should exclude:

* SQL database dumps
* Database backups
* Development configuration files
* Debugging files
* Installation scripts no longer required
* Version-control metadata
* Temporary files

## Protect Sensitive Backups

Database backups should be stored in access-controlled locations and encrypted when appropriate.

---

# Conclusion

The Vehicle Management System in PHP may expose its `vehicle_management.sql` database file through the publicly accessible application directory. If the application is deployed with the SQL file under the web server document root and direct access to `.sql` resources is permitted, an unauthenticated attacker can retrieve the database file through a simple HTTP GET request.

Disclosure of the SQL file can expose the application's database structure and any data contained within the distributed or deployed database dump. The vulnerability should be remediated by removing SQL files from publicly accessible directories, restricting access to sensitive file extensions, and ensuring that production deployment packages do not contain unnecessary database dumps or backup files.


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/dbfe040a-4c97-485e-8e7c-9cb0a3d51e18" />
