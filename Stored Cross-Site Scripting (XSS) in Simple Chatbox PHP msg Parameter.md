## Stored Cross-Site Scripting (XSS) in Simple Chatbox PHP msg Parameter
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

Stored Cross-Site Scripting (Stored XSS)

CWE

CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-Site Scripting)

## Severity

Medium

CVSS v3.1 Score

6.1 (Medium)

Vector:

CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N
Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Simple Chatbox in PHP within the message submission functionality.

## The vulnerability is located in the following endpoint:

/SimpleChatbox_PHP/chatbox/insert.php

The application processes user-controlled input through the msg parameter supplied via an HTTP GET request. The provided message is stored by the application and later displayed in the chat interface without proper validation or output encoding.

Because the application directly stores and renders user input without sanitization, attackers can inject malicious HTML or JavaScript code into chat messages.

During testing, the following payload was successfully injected:

<script>alert(1)</script>

 injected value:

Your message here...gigcv<script>alert(1)</script>owsxa

Once submitted, the malicious payload is stored in the backend and executed whenever the chat messages are displayed to users.

This confirms that the vulnerability is a Stored (Persistent) Cross-Site Scripting issue.

## Root Cause

The vulnerability is caused by improper handling of user input.

Specifically:

User input is accepted without validation

Data is stored without sanitization

Output is rendered without encoding (e.g., htmlspecialchars)

The application directly inserts user input into HTML content

A typical vulnerable pattern:

$msg = $_GET['msg'];
// stored in database or file
echo $msg; // rendered without encoding

Because the input is not sanitized or encoded, malicious scripts are executed by the browser.

## Affected Endpoint
/SimpleChatbox_PHP/chatbox/insert.php
## Vulnerable Parameter
msg
## HTTP Method
GET
## Proof of Concept
Request
GET /SimpleChatbox_PHP/chatbox/insert.php?msg=Hello<script>alert(1)</script> HTTP/1.1
Host: localhost

Captured request:

GET /SimpleChatbox_PHP/chatbox/insert.php?msg=Your%20message%20here...gigcv%3cscript%3ealert(1)%3c%2fscript%3eowsxa HTTP/1.1
Host: localhost
Steps to Reproduce

Install the Simple Chatbox in PHP.

Open the chat interface.

Intercept or manually craft a request.

Inject the payload in the msg parameter:

<script>alert(1)</script>

Send the request.

Reload or view the chat interface.

Result

The injected JavaScript payload executes when the chat messages are displayed, confirming Stored XSS.

Impact

Successful exploitation allows attackers to:

Execute arbitrary JavaScript in victim browsers

Steal session cookies

Hijack user sessions

Perform actions on behalf of users

Inject malicious or phishing content into the chat

Affect all users viewing the chat (persistent impact)

Vulnerability Classification
CWE

CWE-79 – Cross-Site Scripting

OWASP Top 10

A03:2021 – Injection

Suggested Remediation
Output Encoding
echo htmlspecialchars($msg, ENT_QUOTES, 'UTF-8');
Input Validation

Sanitize user input before storing it.

Content Security Policy
Content-Security-Policy: default-src 'self'

## PoC

<img width="1505" height="890" alt="Image" src="https://github.com/user-attachments/assets/485f19b0-12b2-42ef-b5f3-8f4f64162891" />
