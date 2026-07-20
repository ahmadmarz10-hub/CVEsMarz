# Hotel and Tourism Reservation System – Sensitive Information Disclosure via Exposed SQL Database File

## Credit

**Discovered by:** Ahmad Marzouk

## Product

Hotel and Tourism Reservation in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/hotel-and-tourism-reservation-in-php-with-source-code/

## Affected Version

Version not specified.

The exact affected version should be confirmed from the tested source-code package before formal CVE or VulDB submission.

## Vulnerability Type

Sensitive Information Disclosure / Exposed SQL Database File

## CWE

**CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor**

## OWASP Top 10

**A05:2021 – Security Misconfiguration**

## Severity

**High**

## Suggested CVSS v3.1 Score

**7.5 (High)**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
```

The final CVSS score depends on the contents of the exposed SQL file and whether the resource is remotely accessible without authentication.

---

# Vulnerability Overview

The Hotel and Tourism Reservation System in PHP is affected by a potential Sensitive Information Disclosure vulnerability caused by an SQL database file being stored within a web-accessible application directory.

The affected resource is:

```text
/ht/hotel_db%20(1).sql
```

The decoded filename is:

```text
hotel_db (1).sql
```

If the application is deployed under a web server document root and the server permits direct access to `.sql` files, an unauthenticated attacker may be able to retrieve the database file directly through an HTTP GET request.

The tested local resource is:

```text
https://localhost/ht/hotel_db%20(1).sql
```

The presence of `%20` in the URL represents a space character in the underlying filename.

If direct retrieval is successful, the exposed SQL file may disclose database schema information and any application data included in the database dump.

---

# Description

The Hotel and Tourism Reservation System in PHP contains a potential Sensitive Information Disclosure vulnerability due to an SQL database file being accessible from within the application's web directory.

The affected database resource is named:

```text
hotel_db (1).sql
```

and is accessible using the URL-encoded path:

```text
/ht/hotel_db%20(1).sql
```

Database initialization files and database backups commonly contain SQL statements used to create application tables and populate them with data. These resources should not be accessible through the public web server.

If the `hotel_db (1).sql` file is located under the application's publicly accessible `/ht/` directory and the web server does not explicitly restrict requests for `.sql` files, an unauthenticated remote user can potentially request the resource directly.

The attack does not require exploitation of the application's normal reservation functionality. Instead, the attacker sends a direct HTTP GET request for the predictable database filename.

If the server responds with the SQL file, the attacker can obtain information contained within the database dump.

Depending on the contents of the exposed file, disclosed information may include:

* Database table names
* Database column names
* Application database structure
* User account records
* Administrative account information
* Hotel information
* Room information
* Reservation records
* Customer information
* Contact information
* Tour information
* Other records contained in the database dump

Even when an SQL file contains only the initial application schema and demonstration data, disclosure of the complete database structure provides an attacker with detailed knowledge of the application's backend implementation.

This information can reveal table relationships, authentication-related fields, administrative data structures, and other implementation details that may assist in identifying or exploiting additional application vulnerabilities.

If the SQL file contains credentials, password hashes, plaintext passwords, personal information, or real reservation data, the confidentiality impact is substantially higher.

The root cause is insecure placement of a database resource inside a publicly accessible web directory combined with insufficient web-server access restrictions.

---

# Root Cause

The vulnerability results from insecure storage and deployment of a database SQL file.

Potential contributing factors include:

* An SQL database file is stored within the application directory.
* The application directory is deployed under the web server document root.
* The SQL file remains present after installation.
* The web server permits direct requests for `.sql` resources.
* No authentication mechanism protects the static SQL resource.
* No server-level rule prevents access to database files.
* Sensitive installation resources are not removed before production deployment.

As a result, a database file may be retrievable through a direct HTTP request.

---

# Affected Resource

```text
/ht/hotel_db%20(1).sql
```

Decoded filesystem-style filename:

```text
hotel_db (1).sql
```

## HTTP Method

```text
GET
```

## Authentication Required

```text
None
```

assuming the file is directly accessible as reported.

---

# Proof of Concept

## Full HTTP Request

```http
GET /ht/hotel_db%20(1).sql HTTP/1.1
Host: localhost
Accept: */*
Connection: close
```

## Tested Local Resource

```text
https://localhost/ht/hotel_db%20(1).sql
```

If the application is deployed over plain HTTP instead of HTTPS, the equivalent request would target the same `/ht/hotel_db%20(1).sql` path over HTTP.

The vulnerability is confirmed if the server responds successfully and returns or downloads the contents of the SQL file without requiring authentication.

---

# Steps to Reproduce

1. Deploy the Hotel and Tourism Reservation System in PHP in an authorized local testing environment.

2. Ensure the complete application package is deployed under the web server document root.

3. Open a browser or HTTP client.

4. Request the following resource:

```text
/ht/hotel_db%20(1).sql
```

5. Alternatively, send:

```http
GET /ht/hotel_db%20(1).sql HTTP/1.1
Host: localhost
Connection: close
```

6. Observe the HTTP response.

7. If the server responds with the contents of `hotel_db (1).sql` or offers the file for download without requiring authentication, the information disclosure vulnerability is confirmed.

---

# Expected Result

Database initialization files, SQL dumps, and backup files should not be accessible through HTTP.

A direct request for:

```text
/ht/hotel_db%20(1).sql
```

should return an appropriate response such as:

```text
403 Forbidden
```

or:

```text
404 Not Found
```

The SQL file should preferably not exist anywhere within the production web document root.

---

# Observed Result

The reported affected resource is:

```text
https://localhost/ht/hotel_db%20(1).sql
```

If requesting this resource returns the SQL database file without authentication, an unauthenticated user can retrieve potentially sensitive database information.

The final vulnerability submission should include the observed HTTP status code and a small, redacted portion of the response demonstrating that the returned resource is an SQL database file.

---

# Attack Scenario

A typical attack scenario is:

1. An application administrator deploys the complete Hotel and Tourism Reservation project under the web server document root.

2. The SQL database file remains inside the `/ht/` directory.

3. The web server is configured to serve unknown or static file extensions directly.

4. An unauthenticated attacker identifies or predicts the SQL filename.

5. The attacker sends a direct GET request for:

```text
/ht/hotel_db%20(1).sql
```

6. The web server returns the database file.

7. The attacker analyzes the disclosed database structure and any stored records contained in the dump.

No application-level authentication or user interaction is required if the static resource is publicly accessible.

---

# Security Impact

Successful exploitation may disclose sensitive information contained within the SQL file.

Potentially exposed information includes:

* Complete database schema
* Database table and column names
* Administrative account records
* User account information
* Customer records
* Reservation information
* Hotel and room records
* Tour information
* Contact information
* Password hashes or credentials if included in the dump
* Other application data present in the SQL file

Disclosure of this information may assist further attacks against the application.

If valid credentials are contained in the SQL file, attackers may potentially attempt unauthorized access to affected accounts.

The exact impact depends on the actual contents of `hotel_db (1).sql`.

---

# Vulnerability Classification

## CWE

**CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor**

Depending on the precise nature of the exposed resource, the issue may also be associated with insecure deployment or configuration practices.

## OWASP Top 10

**A05:2021 – Security Misconfiguration**

---

# Remediation

## Remove SQL Files from the Web Root

The following resource should not be present in the publicly accessible application directory:

```text
hotel_db (1).sql
```

Remove database initialization files after installation.

---

## Store Database Files Outside the Document Root

If database dumps must be retained, store them in a protected directory that cannot be accessed through HTTP.

For example:

```text
/var/backups/hotel-reservation/
```

---

## Block Access to SQL Files

Configure the web server to reject requests for SQL resources.

Apache example:

```apache
<FilesMatch "\.(sql|dump|bak|backup)$">
    Require all denied
</FilesMatch>
```

Nginx example:

```nginx
location ~* \.(sql|dump|bak|backup)$ {
    deny all;
    return 403;
}
```

---

## Harden Production Deployments

Production releases should exclude unnecessary development and installation resources, including:

* SQL database files
* Database backups
* Installation scripts
* Debug files
* Temporary files
* Development configuration files
* Version-control metadata

---

## Review Potentially Exposed Information

If `hotel_db (1).sql` was publicly accessible, administrators should inspect the file to determine what information was potentially disclosed.

If credentials are present, affected passwords and secrets should be rotated.

If personal or reservation information is included, administrators should evaluate the exposure according to applicable data-protection requirements.

---

# Conclusion

The Hotel and Tourism Reservation System in PHP may expose the `hotel_db (1).sql` database file through the publicly accessible `/ht/` application directory.

The affected resource can be represented through the URL-encoded path `/ht/hotel_db%20(1).sql`. If the web server returns this file without requiring authentication, an unauthenticated attacker can retrieve the database resource directly.

Successful exploitation may expose the application's database schema and any records contained within the SQL dump, potentially including account, customer, hotel, room, and reservation information.

If direct unauthenticated retrieval has been confirmed, the issue should be classified as **Sensitive Information Disclosure under CWE-200**. The SQL file should be removed from the web root, database backups should be stored in protected locations, and the web server should explicitly deny access to database-related file extensions.



<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/66cd2a62-c17e-45b6-8085-f3cc5a2248f6" />
