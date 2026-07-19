# Daily Expense Manager in PHP – Sensitive Information Disclosure via Exposed SQL Database File

## Credit

**Discovered by:** Ahmad Marzook

## Product

Daily Expense Manager in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/daily-expense-manager-in-php-with-source-code/

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

This score assumes that the SQL file is remotely accessible without authentication and contains sensitive database information. The final score should be adjusted according to the actual contents of the exposed file and deployment configuration.

# Vulnerability Overview

The Daily Expense Manager in PHP is affected by a potential Sensitive Information Disclosure vulnerability caused by an SQL database file being stored within the application's web-accessible directory.

The affected resource is:

```text
/Daily-Expense-Manager/exp_ak.sql
```

The application's installation procedure requires the `exp_ak.sql` database file to initialize the application's MySQL database. The file is located inside the `Daily-Expense-Manager` project directory.

If an administrator deploys the entire project directory under the web server document root without removing the SQL file or configuring appropriate access restrictions, the database file may be directly accessible over HTTP.

An unauthenticated attacker could potentially retrieve the resource by requesting:

```text
http://localhost/Daily-Expense-Manager/exp_ak.sql
```

On a publicly deployed installation, `localhost` would be replaced by the hostname or IP address of the affected server.

If the web server permits direct access to `.sql` files, the attacker may receive the database dump without interacting with the application's normal functionality or authentication mechanisms.

# Description

The Daily Expense Manager in PHP contains a potential Sensitive Information Disclosure vulnerability due to the insecure placement of the `exp_ak.sql` database initialization file within the application's web directory.

The SQL file is distributed as part of the application and is used to create and initialize the application's MySQL database. The vendor's documented installation process instructs administrators to copy the main project directory into the XAMPP `htdocs` directory and subsequently import the `exp_ak.sql` file located inside the `Daily-Expense-Manager` folder.

As a consequence, following the documented deployment procedure may leave the SQL file inside the web server's document root.

The affected resource is:

```text
/Daily-Expense-Manager/exp_ak.sql
```

If the web server is configured to serve `.sql` files as static resources and no additional access restrictions are implemented, an unauthenticated user can request the file directly through HTTP.

The vulnerability does not require interaction with an authenticated application endpoint. Instead, the attacker accesses the static database resource directly using its predictable URL.

Depending on the contents of the deployed SQL file, successful exploitation could disclose the application's complete database schema and any records contained in the database dump. Potentially exposed information may include income records, expense information, financial transaction details, database table and column names, and other application data present in the SQL file.

Even if the distributed `exp_ak.sql` file contains only initial or sample data rather than current production information, exposing the complete database schema provides an attacker with detailed knowledge of the application's backend structure. This information can assist in identifying table names, column names, relationships, and application data structures that may facilitate additional attacks.

The root cause is the placement of a sensitive database initialization file inside a publicly accessible application directory combined with the absence of web-server restrictions preventing direct access to SQL resources.

If direct HTTP retrieval of `exp_ak.sql` is confirmed, exploitation requires no authentication, privileges, or user interaction.

# Root Cause

The vulnerability results from insecure deployment and storage of the application's database initialization file.

Contributing factors include:

* The `exp_ak.sql` file is distributed inside the main application directory.
* Installation instructions direct administrators to place the project directory under the web server document root.
* The SQL file may remain in the application directory after database initialization.
* Direct access to `.sql` resources may not be explicitly blocked by the web server.
* No application-level authentication protects static database files.
* Sensitive installation resources are not automatically removed following installation.

As a result, the SQL database file may become accessible through a predictable HTTP path.

# Affected Resource

```text
/Daily-Expense-Manager/exp_ak.sql
```

## HTTP Method

```text
GET
```

# Proof of Concept

## Full HTTP Request

```http
GET /Daily-Expense-Manager/exp_ak.sql HTTP/1.1
Host: localhost
Accept: */*
Connection: close
```

## Tested Local Resource

```text
http://localhost/Daily-Expense-Manager/exp_ak.sql
```

If the server responds with the contents of `exp_ak.sql` or offers the file for download without requiring authentication, the information disclosure vulnerability is confirmed.

# Steps to Reproduce

1. Deploy the Daily Expense Manager in PHP in an authorized local testing environment.

2. Copy the `Daily-Expense-Manager` application folder into the web server document root.

3. Import `exp_ak.sql` into the application's MySQL database as required during installation.

4. Leave the original SQL file in the deployed application directory.

5. Send the following request:

```http
GET /Daily-Expense-Manager/exp_ak.sql HTTP/1.1
Host: localhost
Connection: close
```

6. Observe the server response.

7. If the server returns the SQL database file without requiring authentication, the vulnerability is confirmed.

# Expected Result

Sensitive database initialization and backup files should not be publicly accessible through the web server.

A direct request for the affected resource should result in an error such as:

```text
403 Forbidden
```

or:

```text
404 Not Found
```

# Observed Result

The following resource is reported as directly accessible:

```text
/Daily-Expense-Manager/exp_ak.sql
```

If the web server returns the SQL contents without authentication, an unauthenticated user can retrieve the exposed database resource.

# Impact

Successful exploitation may disclose information contained within the SQL file, potentially including:

* Database schema information
* Database table and column names
* Income records
* Expense records
* Financial information stored in the dump
* Application configuration data stored in database tables
* Other application records included in the SQL file

Disclosure of the database structure may also assist an attacker in understanding the application's internal implementation and planning additional attacks.

The actual confidentiality impact depends on whether the exposed file contains only the application's initial database structure and sample records or includes real data from the affected deployment.

# Authentication Requirements

If the SQL file is publicly accessible:

```text
Attack Vector: Network
Privileges Required: None
User Interaction: None
```

No application account is required because the attacker accesses the static SQL resource directly.

# Vulnerability Classification

**CWE**

CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

**OWASP Top 10**

A05:2021 – Security Misconfiguration

# Remediation

## Remove SQL Files from the Web Root

Database initialization files and database backups should not be stored within publicly accessible directories.

After installation, remove:

```text
exp_ak.sql
```

from the deployed web application.

## Store Database Files Outside the Document Root

If the SQL file must be retained, store it in a directory that cannot be accessed through HTTP.

For example:

```text
/var/backups/daily-expense-manager/
```

## Restrict Access to SQL Resources

As a defense-in-depth measure, configure the web server to reject requests for database-related files.

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

## Harden the Deployment Process

Production deployments should exclude unnecessary sensitive resources, including:

* SQL database dumps
* Database backup files
* Installation files
* Development configuration files
* Debugging resources
* Temporary files
* Version-control metadata

## Review Exposed Data

If the SQL file has previously been publicly accessible, administrators should review its contents to determine what information may have been exposed.

Any credentials contained in the file should be considered compromised and rotated.

# Conclusion

The Daily Expense Manager in PHP may expose its `exp_ak.sql` database initialization file when the complete application directory is deployed under the web server document root. The documented installation procedure places the project directory under `htdocs` and identifies `exp_ak.sql` as being located inside the `Daily-Expense-Manager` folder.

If the SQL file remains in this location and the web server permits direct access to `.sql` resources, an unauthenticated attacker can potentially retrieve the database file through a direct HTTP GET request.

Successful exploitation may expose the application's database schema and any data contained within the SQL dump. Developers and administrators should remove database initialization files after installation, store backups outside the document root, and configure the web server to deny direct access to sensitive database resources.


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/e7fdc89f-8dc1-43af-bb70-b3f4848f2189" />
