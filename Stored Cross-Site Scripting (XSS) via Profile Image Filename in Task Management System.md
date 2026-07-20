# Stored Cross-Site Scripting (XSS) via Profile Image Filename in Task Management System

## Credit

**Discovered by:** Ahmad Marzouk

---

## Product

Task Management System in PHP

---

## Vendor

Code-Projects

---

## Vendor URL

https://code-projects.org/task-management-system-in-php-with-source-code/

---

## Affected Version

Version 1.0

---

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS) via Uploaded Filename

---

## CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

Potentially related:

**CWE-434 – Unrestricted Upload of File with Dangerous Type**

CWE-434 should only be included if the application also fails to enforce appropriate file-type restrictions and allows uploaded files to be stored in a security-sensitive location.

---

## OWASP Top 10

**A03:2021 – Injection**

---

## Severity

**Medium**

---

## Suggested CVSS v3.1 Score

**5.4 (Medium)**

```text
CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N
```

The final score depends on where the filename is rendered and whether privileged administrators or other users can view the affected uploaded-file information.

---

# Vulnerability Overview

The Task Management System in PHP contains a potential Stored Cross-Site Scripting vulnerability in the profile picture update functionality implemented by:

```text
POST /PROJECT_NEW/user/UpdateUserProfile.php
```

The affected multipart upload field is:

```text
profile
```

An authenticated user can submit a file whose filename contains attacker-controlled HTML markup.

The provided request uses the following malicious filename:

```html
file.txtp5vvr"><img src=a onerror=alert(1)>pshx9
```

The injected XSS element is:

```html
<img src=a onerror=alert(1)>
```

If the application stores the original uploaded filename and subsequently inserts it into an HTML document without applying contextual output encoding, the browser may interpret the injected `<img>` element as HTML.

Because the supplied image source is invalid, the browser triggers the `onerror` event handler, causing the embedded JavaScript to execute.

If the malicious filename remains stored and executes whenever a profile, upload record, administrative user page, or related interface is viewed, the issue constitutes Stored Cross-Site Scripting.

---

# Description

The Task Management System in PHP is potentially vulnerable to Stored Cross-Site Scripting through attacker-controlled filenames supplied to the user profile upload functionality.

The affected endpoint processes multipart/form-data requests containing a `profile` file field. The filename component of a multipart upload is fully controlled by the client and therefore must be considered untrusted input.

The supplied proof-of-concept request specifies the following filename:

```html
file.txtp5vvr"><img src=a onerror=alert(1)>pshx9
```

This value contains injected HTML markup with an event-handler attribute:

```html
<img src=a onerror=alert(1)>
```

If the application saves the original filename in the database or filesystem metadata and later displays that filename within an HTML response without proper encoding, the injected markup may break out of the intended HTML context.

The browser then parses the injected `<img>` element as active HTML. Because the specified image source cannot be loaded, the element's `onerror` event handler is triggered and the supplied JavaScript executes in the security context of the Task Management System.

An attacker could exploit the issue by authenticating with a valid account, uploading a profile file containing a malicious filename, and causing another user to access a page that displays the stored filename.

Potentially affected locations may include:

* User profile pages
* Administrative user-management interfaces
* Profile image management pages
* Upload history pages
* Account listings
* Task or project member interfaces

The exact execution location should be identified in the final vulnerability report based on where the payload was observed to execute.

Because the malicious value originates in the filename rather than the contents of the uploaded file, restricting only the uploaded file's MIME type does not prevent this vulnerability. Filenames must independently be treated as attacker-controlled data and safely encoded whenever displayed.

If the filename is stored persistently and executes during subsequent page views, this issue should be classified as Stored Cross-Site Scripting under CWE-79.

---

# Root Cause

The vulnerability is caused by unsafe handling of an attacker-controlled uploaded filename.

Potential contributing factors include:

* Trusting the client-supplied multipart filename.
* Retaining the original filename without normalization.
* Storing the original filename in the application's database.
* Rendering the filename directly into HTML.
* Failure to apply contextual HTML output encoding.
* Insufficient filename validation.

A vulnerable implementation may resemble:

```php
$filename = $_FILES['profile']['name'];

move_uploaded_file(
    $_FILES['profile']['tmp_name'],
    'uploads/' . $filename
);

echo $filename;
```

The XSS vulnerability arises when the attacker-controlled `$filename` is inserted directly into an HTML response.

For example:

```php
echo "<p>Uploaded file: " . $filename . "</p>";
```

A safer output pattern is:

```php
echo htmlspecialchars(
    $filename,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

---

# Affected Endpoint

```text
POST /PROJECT_NEW/user/UpdateUserProfile.php
```

---

# Vulnerable Multipart Field

```text
profile
```

---

# Attacker-Controlled Input

```text
filename
```

---

# Supporting Parameter

```text
updatePic
```

---

# Authentication Requirements

The supplied request contains a valid PHP session cookie, indicating that access to the profile update functionality likely requires authentication.

Likely requirements:

```text
Privileges Required: Low
User Interaction: Required
```

The attacker must be able to submit a profile upload, and another user must subsequently access a page where the malicious filename is rendered.

---

# Proof of Concept

## Full HTTP Request

```http
POST /PROJECT_NEW/user/UpdateUserProfile.php HTTP/1.1
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
Cookie: PHPSESSID=b8ojnmipqs37i0vbo0khqe9q7p
Origin: https://localhost
Upgrade-Insecure-Requests: 1
Referer: https://localhost/PROJECT_NEW/user/UpdateUserProfile.php
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryHKMFv2k16P9B8U3r
Content-Length: 291

------WebKitFormBoundaryHKMFv2k16P9B8U3r
Content-Disposition: form-data; name="profile"; filename="file.txtp5vvr\"><img src=a onerror=alert(1)>pshx9"
Content-Type: text/plain

933GCJdmer
------WebKitFormBoundaryHKMFv2k16P9B8U3r
Content-Disposition: form-data; name="updatePic"


------WebKitFormBoundaryHKMFv2k16P9B8U3r--
```

---

# Malicious Filename

```html
file.txtp5vvr"><img src=a onerror=alert(1)>pshx9
```

The relevant HTML injection component is:

```html
<img src=a onerror=alert(1)>
```

---

# Steps to Reproduce

1. Deploy the Task Management System in PHP in an authorized local testing environment.

2. Authenticate using a valid test account.

3. Navigate to the user profile update functionality:

```text
/PROJECT_NEW/user/UpdateUserProfile.php
```

4. Intercept the profile picture upload request using an HTTP testing proxy.

5. Modify the multipart `filename` value for the `profile` field so that it contains HTML markup.

6. Submit the modified upload request.

7. Navigate to any page that displays the uploaded filename or associated profile information.

8. Observe whether the browser interprets the stored filename as HTML.

9. Reload the affected page or access it with another authorized test account.

10. If the payload remains present and executes during later requests, Stored XSS is confirmed.

---

# Expected Result

The application should treat all uploaded filenames exclusively as untrusted text.

A filename containing characters such as:

```text
< > " '
```

must never be interpreted as HTML or JavaScript when displayed.

The application should either generate a server-side filename or safely encode the original name before displaying it.

---

# Observed Result

The supplied request demonstrates that the multipart upload permits an attacker-controlled filename containing HTML and JavaScript event-handler markup.

If this filename is stored and later rendered without contextual output encoding, the injected HTML executes within the browser.

The final report should document the exact page where execution was observed.

---

# Impact

Successful exploitation of confirmed Stored XSS may allow an authenticated attacker to execute arbitrary JavaScript in the browser of users who view the affected stored filename.

Potential consequences include:

* Performing unauthorized actions using a victim's active session.
* Modifying application content displayed to the victim.
* Accessing information available to client-side scripts.
* Injecting deceptive or phishing interfaces.
* Redirecting users to attacker-controlled pages.
* Targeting administrators who review user profiles or uploads.
* Persistently affecting multiple users until the malicious record is removed.

The practical severity depends on which users can view the filename and the privileges held by those users.

---

# Additional File Upload Security Considerations

The supplied upload also uses:

```text
Content-Type: text/plain
```

for a field apparently intended to contain a profile image.

If the application accepts arbitrary file formats instead of restricting uploads to expected image types, this may indicate a separate file-upload security weakness.

This should be tested and reported separately from the filename-based XSS issue.

A file-upload vulnerability should only be classified as CWE-434 if testing confirms that dangerous or unintended file types can be uploaded in a way that creates a meaningful security impact.

---

# Vulnerability Classification

## Primary CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**

## Potential Secondary CWE

**CWE-434 – Unrestricted Upload of File with Dangerous Type**

Only applicable if unrestricted file upload is independently confirmed.

## OWASP Top 10

**A03:2021 – Injection**

---

# Remediation

## Generate Server-Side Filenames

Do not rely on or directly use the original client-supplied filename for storage.

Generate a random server-side name instead.

Example:

```php
$extension = strtolower(
    pathinfo($_FILES['profile']['name'], PATHINFO_EXTENSION)
);

$storedName = bin2hex(random_bytes(16)) . '.' . $extension;
```

---

## Encode Filenames Before Rendering

Whenever an original filename must be displayed, apply contextual HTML encoding:

```php
echo htmlspecialchars(
    $originalFilename,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

---

## Validate Upload Type

For a profile picture upload, restrict accepted files to explicitly supported image formats.

Do not trust the client-supplied `Content-Type` header alone.

Inspect the actual file type using server-side mechanisms such as:

```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->file($_FILES['profile']['tmp_name']);
```

Allow only expected MIME types.

---

## Validate File Extensions

Use an allowlist of supported extensions, such as:

```text
.jpg
.jpeg
.png
.webp
```

File extension validation should be combined with MIME and content validation.

---

## Store Uploads Safely

Uploaded files should ideally be stored:

* Outside executable web directories.
* Under server-generated filenames.
* Without execute permissions.
* With appropriate filesystem permissions.

---

## Implement Content Security Policy

A restrictive Content Security Policy can provide defense in depth against certain XSS attacks.

Example:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'
```

CSP does not replace proper output encoding.

---

# Conclusion

The Task Management System in PHP contains a potential Stored Cross-Site Scripting vulnerability in the profile upload functionality of `/PROJECT_NEW/user/UpdateUserProfile.php`.

An authenticated user can control the multipart filename associated with the `profile` upload field and supply HTML event-handler markup within that filename. If the application stores this value and later displays it without contextual output encoding, the injected markup may execute as JavaScript in the browser of users viewing the affected record.

If persistence and subsequent JavaScript execution have been confirmed, the vulnerability should be classified as **Stored Cross-Site Scripting under CWE-79**.

The application should generate safe server-side filenames, encode original filenames before display, validate actual uploaded file types, and review the upload functionality for additional unrestricted file-upload weaknesses.


<img width="1512" height="982" alt="Image" src="https://github.com/user-attachments/assets/987cc2bf-61cd-4a79-8cbb-a589243bed55" />
