# Reflected Cross-Site Scripting (XSS) in Hotel and Tourism Reservation System `room` Parameter

## Credit

**Discovered by:** Ahmad Marzouk

---

## Product

Hotel and Tourism Reservation in PHP

---

## Vendor

Code-Projects

---

## Vendor Homepage

https://code-projects.org/hotel-and-tourism-reservation-in-php-with-source-code/

---

## Affected Version

Version not specified.

The exact affected version should be confirmed from the tested source-code package before formal CVE or VulDB submission.

---

## Vulnerability Type

Reflected Cross-Site Scripting (Reflected XSS)

---

## CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

---

## OWASP Top 10

**A03:2021 – Injection**

---

## Severity

**Medium**

---

## Suggested CVSS v3.1 Score

**6.1 (Medium)**

```text id="tr2vhl"
CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N
```

This score assumes that the vulnerable reservation page is accessible without authentication and that exploitation requires a victim to open a crafted request.

---

# Vulnerability Overview

The Hotel and Tourism Reservation System in PHP contains a potential Reflected Cross-Site Scripting vulnerability in the room reservation functionality.

The affected endpoint is:

```text id="fzciwd"
/ht/details.php
```

The vulnerability affects the following GET parameter:

```text id="j0sgdw"
room
```

The supplied proof-of-concept request places HTML and JavaScript markup into the `room` parameter:

```html id="j53mxv"
"><script>alert(1)</script>
```

If the application includes the supplied `room` value directly in the generated HTML response without contextual output encoding, the browser interprets the injected `<script>` element as executable JavaScript.

Because the malicious value is supplied through the request and reflected immediately into the resulting page rather than being stored persistently in the application database, the vulnerability is classified as Reflected Cross-Site Scripting.

---

# Description

The Hotel and Tourism Reservation System in PHP is vulnerable to a potential Reflected Cross-Site Scripting vulnerability in the `room` parameter processed by the `details.php` endpoint.

The affected page accepts a room identifier through the URL query string and uses this value while displaying room information or processing reservation-related functionality.

The application appears to accept arbitrary user-controlled characters in the `room` parameter. The supplied proof-of-concept value contains characters designed to terminate an existing HTML context followed by an executable `<script>` element.

The decoded malicious value is:

```html id="xdh4ga"
15hcpdj"><script>alert(1)</script>g2whhcd19l7
```

The relevant injected JavaScript component is:

```html id="do6e6e"
<script>alert(1)</script>
```

If the application directly inserts the supplied `room` parameter into an HTML response without encoding special characters such as `<`, `>`, `"`, and `'`, an attacker-controlled value can alter the structure of the generated HTML document.

When a victim follows a specially crafted URL containing the malicious `room` value, the application reflects the payload into the response. The victim's browser then interprets the injected markup as executable content and runs the JavaScript in the security context of the affected application.

Unlike Stored XSS, the malicious value does not necessarily remain in the application's database. Exploitation typically requires the attacker to convince a victim to visit a specially crafted URL or otherwise initiate the affected request.

Successful exploitation may allow arbitrary JavaScript execution within the victim's browser session. Depending on the victim's authentication state and the application's security controls, an attacker may potentially perform unauthorized actions using the victim's active session, alter displayed content, access information available to client-side scripts, or display deceptive content within the trusted application origin.

The vulnerability results from the application's failure to apply proper contextual output encoding to user-controlled URL parameters before including them in dynamically generated HTML.

---

# Root Cause

The vulnerability is caused by unsafe rendering of attacker-controlled input.

Potential contributing factors include:

* Direct use of the `room` GET parameter in dynamically generated HTML.
* Failure to apply contextual output encoding.
* Insufficient server-side validation of the expected room identifier.
* Trusting URL parameters as safe HTML content.
* Allowing unexpected HTML metacharacters in an identifier field.

A vulnerable implementation could resemble:

```php id="7odg5i"
$room = $_GET['room'];

echo '<input type="text" value="' . $room . '">';
```

With attacker-controlled input, the quotation mark can terminate the `value` attribute and allow new HTML elements to be introduced.

Another unsafe pattern would be:

```php id="awtw8c"
echo $_GET['room'];
```

The exact vulnerable source-code location should be reviewed and included in the final advisory where available.

---

# Affected Endpoint

```text id="1z3n96"
GET /ht/details.php
```

---

# Vulnerable Parameter

```text id="e0yjuq"
room
```

---

# Additional Parameters

The supplied request also contains:

```text id="swcirg"
fullname
in_date
out_date
phone
people
rooms
email
checkin
```

The provided evidence identifies `room` as the tested XSS parameter.

---

# HTTP Method

```text id="m59d48"
GET
```

---

# Proof of Concept

## Full HTTP Request

```http id="5fqzr5"
GET /ht/details.php?room=15hcpdj%22%3e%3cscript%3ealert(1)%3c%2fscript%3eg2whhcd19l7&fullname=&in_date=&out_date=&phone=&people=&rooms=&email=&checkin=Make+Reservation HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Sec-CH-UA: "Google Chrome";v="146", "Not=A?Brand";v="8", "Chromium";v="146"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "macOS"
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Connection: close
Cookie: PHPSESSID=jc7rpf4pjkpa5d6mmi40thnj06
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/ht/details.php?room=15
```

---

# Decoded Request Target

```text id="tbj40g"
/ht/details.php?room=15hcpdj"><script>alert(1)</script>g2whhcd19l7&fullname=&in_date=&out_date=&phone=&people=&rooms=&email=&checkin=Make+Reservation
```

---

# Test Payload

```html id="eb0l5t"
"><script>alert(1)</script>
```

---

# Steps to Reproduce

1. Deploy the Hotel and Tourism Reservation System in PHP in an authorized local testing environment.

2. Navigate to a valid room details page, for example:

```text id="644tu6"
/ht/details.php?room=15
```

3. Intercept or modify the request.

4. Replace the normal `room` identifier with a value containing the test HTML/JavaScript input.

5. Send the modified request.

6. Inspect the returned HTML response and determine whether the submitted `room` value is reflected.

7. Open the response in a browser.

8. If the reflected `<script>` element is interpreted as HTML and the JavaScript executes, Reflected Cross-Site Scripting is confirmed.

---

# Expected Result

The application should treat the `room` parameter exclusively as a room identifier.

Because this value appears to represent a numeric room ID, malformed values should be rejected before processing.

For example, values containing HTML markup should never be inserted into an HTML response as executable content.

---

# Observed Result

The supplied request introduces HTML and JavaScript syntax through the `room` query parameter.

If the application reflects this value into the generated HTML without encoding and the browser executes the injected script, the behavior confirms a Reflected Cross-Site Scripting vulnerability.

The final vulnerability submission should include the exact response location in which the payload was reflected.

---

# Attack Scenario

A typical attack scenario is:

1. An attacker creates a crafted URL containing malicious input in the `room` parameter.
2. The attacker causes a victim to open the crafted request.
3. The vulnerable application receives the malicious parameter.
4. The application reflects the value into the HTML response without proper encoding.
5. The victim's browser interprets the injected markup as executable JavaScript.
6. The JavaScript executes within the origin of the Hotel and Tourism Reservation application.

User interaction is therefore required for exploitation.

---

# Impact

Successful exploitation may allow an attacker to execute arbitrary JavaScript within the browser of a victim who accesses a maliciously crafted request.

Potential consequences include:

* Performing unauthorized application actions through the victim's active session.
* Modifying application content displayed to the victim.
* Accessing information exposed to client-side JavaScript.
* Creating deceptive or phishing content within the application's trusted origin.
* Redirecting the victim to attacker-controlled content.
* Targeting authenticated administrative users if they can be induced to access a malicious request.

The practical severity depends on the application's session protections, Content Security Policy, authentication model, and the privileges of the targeted victim.

---

# Authentication Requirements

If the room details page is publicly accessible:

```text id="0ozo8n"
Privileges Required: None
User Interaction: Required
```

The attacker does not require an authenticated account but must cause a victim to initiate the crafted request.

---

# Vulnerability Classification

## CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

## OWASP Top 10

**A03:2021 – Injection**

---

# Remediation

## Apply Contextual Output Encoding

All user-controlled values must be encoded before insertion into HTML.

For example:

```php id="od7efh"
$room = $_GET['room'] ?? '';

echo htmlspecialchars(
    $room,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

Encoding should match the specific output context.

---

## Validate the Room Identifier

If `room` is expected to contain a numeric database identifier, enforce strict integer validation.

For example:

```php id="5cwm1k"
$room = filter_input(
    INPUT_GET,
    'room',
    FILTER_VALIDATE_INT
);

if ($room === false || $room === null) {
    http_response_code(400);
    exit('Invalid room identifier.');
}
```

This ensures that unexpected HTML or script markup cannot be used as a valid room identifier.

Input validation should supplement, not replace, output encoding.

---

## Use Prepared Statements

If the `room` parameter is also used in database queries, parameterized statements should be used.

Example:

```php id="ozpb5r"
$stmt = $conn->prepare(
    'SELECT * FROM rooms WHERE id = ?'
);

$stmt->bind_param('i', $room);
$stmt->execute();
```

Prepared statements protect against SQL injection but do not replace output encoding for XSS prevention.

---

## Implement Content Security Policy

A restrictive Content Security Policy can provide additional defense against script injection.

For example:

```http id="r9sjrh"
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'
```

CSP should be used as defense in depth rather than the primary XSS mitigation.

---

## Review Other Query Parameters

Developers should review all user-controlled parameters processed by `details.php`, including:

```text id="635pbx"
fullname
in_date
out_date
phone
people
rooms
email
checkin
```

Any values rendered into HTML should be contextually encoded, and values used in database operations should be passed through parameterized queries.

---

# Conclusion

The Hotel and Tourism Reservation System in PHP contains a potential Reflected Cross-Site Scripting vulnerability in the `room` parameter of `/ht/details.php`.

The supplied proof-of-concept request injects a closing quotation mark followed by a JavaScript `<script>` element into the room identifier. If the application reflects this input into the generated HTML response without proper contextual output encoding, the victim's browser executes the injected JavaScript.

If reflection and execution have been confirmed, the issue should be classified as **Reflected Cross-Site Scripting under CWE-79**.

The vulnerability should be remediated by strictly validating the `room` parameter as the expected data type, applying contextual output encoding whenever user-controlled values are rendered, using parameterized database queries, and deploying a restrictive Content Security Policy as an additional defense.



<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/7a686f28-20cf-41b1-8aaf-c8a28ec46939" />
