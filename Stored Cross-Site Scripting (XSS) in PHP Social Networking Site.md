## Stored Cross-Site Scripting (XSS) in PHP Social Networking Site Post Content

## Credit

Discovered by: Ahmad Marzook

## Product

Social Networking Site in PHP

## Vendor

Code-Projects

## Vendor URL

https://code-projects.org/social-networking-site-in-php-with-source-code-2/

## Affected Version

1.0

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

## Description

The Social Networking Site in PHP 1.0 application is vulnerable to a Stored Cross-Site Scripting (XSS) vulnerability in the post content rendering functionality.

The application allows authenticated users to create and publish posts that appear on the social feed. The post content is stored in the backend database and displayed to other users through the feed page.

However, the application fails to properly sanitize or encode user-controlled input before rendering it in the HTML response.

The vulnerable code responsible for rendering posts retrieves user-generated content from the database and outputs it directly into the page:
 ```plain
$query = $conn->query("select * from post LEFT JOIN members on members.member_id = post.member_id order by post_id DESC");

while($row = $query->fetch()){
?>

<div class="alert"><?php echo $row['content']; ?></div>

<?php } ?>
 ```
<img width="790" height="726" alt="Image" src="https://github.com/user-attachments/assets/f155630c-de85-4c86-8041-7fde199c1027" />

Because the content field is printed directly into the HTML response without output encoding, any HTML or JavaScript stored in the database will be interpreted and executed by the browser.

An attacker can exploit this issue by submitting malicious HTML code within the post content field. The injected payload is stored in the database and automatically executed whenever users view the affected page.

Since the application functions as a social networking platform where users frequently view posts, the malicious payload can affect all users who access the feed page.

## Root Cause

The vulnerability occurs due to improper handling of user-controlled input within the application.

Specifically:

The application accepts arbitrary user input when creating posts.

The input is stored in the database without validation or filtering.

When retrieving posts, the application outputs the stored content directly into the HTML page.

The application does not apply output encoding such as htmlspecialchars().

## vulnerable code:

<div class="alert"><?php echo $row['content']; ?></div>

Because of this behavior, HTML elements embedded within the stored value are interpreted by the browser.

The issue arises from:

Lack of input validation for the post content field

Absence of output encoding when rendering stored data

Direct insertion of database values into HTML responses

## Affected Endpoint

Social feed page displaying posts.

## Vulnerable page:

SocialNetworkingSite_PHP/social_networking_site/home.php

## Vulnerable Parameter
content

## Database table:

post

## Proof of Concept

During testing, the following payload was used:

<details/open/ontoggle=prompt(origin)>

This payload uses the HTML details element with an ontoggle event handler to trigger JavaScript execution.

## Steps to Reproduce

Install and run the Social Networking Site in PHP application.

Create a user account or login with an existing account.

Navigate to the Create Post Share your story here.

Enter the following payload as the post content:

<details/open/ontoggle=prompt(origin)>

Submit the post.

Navigate to the feed or timeline page where posts are displayed.

## Result

When the page renders the stored post, the injected payload is interpreted by the browser.

The JavaScript code executes and triggers a prompt displaying the application's origin.

This confirms that the application renders stored user input without proper sanitization or output encoding, resulting in a Stored Cross-Site Scripting vulnerability.

## Impact

Successful exploitation of this vulnerability allows attackers to execute arbitrary JavaScript in victim browsers.

Potential impacts include:

Execution of malicious scripts in user browsers

Hijacking authenticated sessions

Theft of session cookies

Performing actions on behalf of other users

Injecting malicious content into the social feed

Conducting phishing attacks against users

Modifying the application interface

Because the malicious payload is stored in the database, the attack can affect any user who views the compromised post.

## Remediation

Developers should implement several security measures to prevent cross-site scripting vulnerabilities.

Output Encoding

User-controlled data must be encoded before rendering it in HTML.

## fix:

<div class="alert">
<?php echo htmlspecialchars($row['content'], ENT_QUOTES, 'UTF-8'); ?>
</div>
Input Validation

Validate and sanitize user input before storing it in the database.

Restrict or filter potentially dangerous HTML elements.

Content Security Policy (CSP)

Implement a Content Security Policy to reduce the risk of malicious script execution.

## header:

Content-Security-Policy: default-src 'self';
Secure Development Practices

Developers should:

Escape all user-controlled output

Avoid rendering raw HTML from user input

Perform regular security testing

Implement centralized input validation

## Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

## PoC

https://github.com/user-attachments/assets/2e22694d-43b8-4911-acdf-665b06325340
