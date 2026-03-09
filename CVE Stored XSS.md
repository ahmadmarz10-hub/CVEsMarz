## [Stored-XSS] in Tecnick TCExam v16.5.0

## BUG Author:
Ahmad Marzouk
## Product Information:
## Vendor Homepage:
https://www.tecnick.com/

## Software Tested:
https://demos2.softaculous.com/TCExamsu2xliu1gr/admin/code/index.php

## Affected Version: 
v16.5.0 (latest at time of testing)

## Executive Summary

A Stored Cross-Site Scripting (XSS) vulnerability has been identified in Tecnick TCExam v16.5.0, specifically within the Group Management functionality. The flaw allows an attacker to inject persistent malicious JavaScript payloads into the Name parameter when creating a new group.

Once stored, the payload is executed automatically whenever an administrator or any user with group-viewing permissions accesses the affected pages, leading to complete takeover of the victim’s browser session.

## Vulnerability Details

Vulnerability Type: Stored Cross-Site Scripting (XSS)
Severity Level: CRITICAL (CVSS: 9.0)
## Vulnerable Endpoints:

https://demos2.softaculous.com/TCExamplvdftmhcf/admin/code/tce_edit_group.php

https://demos2.softaculous.com/TCExamplvdftmhcf/admin/code/tce_select_users.php

## Vulnerable Parameter:

Name field in Add New Group

## Vector:

HTML Injection using details element with the ontoggle event handler.

## Root Cause

The application fails to:

Sanitize user-supplied input in the Name parameter

Encode user content before rendering it back into the DOM

Restrict HTML5 event attributes
This results in stored executable JavaScript within administrative views.

## Technical Description

By injecting a crafted HTML payload into the Name field while creating a new group, it is possible to trigger JavaScript execution when the admin later loads the group listing or selection page.

The payload abuses:

The details HTML5 tag

The open attribute

The ontoggle event

This combination triggers without user interaction, making it a highly reliable stored-XSS vector.

Payload Used
<details/open/ontoggle=prompt(origin)>

This payload:

Auto-expands due to the open attribute

Fires ontoggle immediately during page rendering

Executes JavaScript as soon as the page loads

## Steps to Reproduce
1. Log in as a user with permission to create groups (Admin-level or similar).
2. Navigate to the Group Management page:
https://demos2.softaculous.com/TCExamplvdftmhcf/admin/code/tce_edit_group.php
3. Create a new group

In the Name field, insert the payload:

<details/open/ontoggle=prompt(origin)>

Submit the form.

4. Log in as an Administrator or navigate to:
https://demos2.softaculous.com/TCExamplvdftmhcf/admin/code/tce_select_users.php
5. View the newly created group
## Observation:

A JavaScript prompt appears displaying the page origin, confirming stored-XSS execution.

Impact

Full account compromise

Session hijacking

CSRF token theft

Privilege escalation

Admin takeover

Forced actions on behalf of the victim

Data leakage or manipulation

Phishing or social engineering attacks inside the application

Stored XSS is highly dangerous because the malicious script persists in the system.

## Suggested Remediation
Immediate

Apply server-side HTML escaping on all output.

Block dangerous tags and event handlers.

Use htmlspecialchars() or equivalent in PHP.

Short-Term

Implement strict input validation:

Reject <, >, ", ', and event attributes.

Use allowlists.

Long-Term

Adopt a templating engine or a modern framework with built-in XSS protection.

Implement a Content Security Policy (CSP).

Sanitize using libraries such as:

HTML Purifier

OWASP Java HTML Sanitizer

Security Recommendations

Enable Content Security Policy (CSP)

Apply the principle of least privilege

Perform regular code reviews and security scanning

Implement a Web Application Firewall (WAF)

Ensure proper logging and monitoring of admin actions

## References

OWASP XSS Prevention Cheat Sheet

CWE-79: Improper Neutralization of Input (Cross-Site Scripting)

CERT Secure Coding Standards

https://github.com/user-attachments/assets/277cbe7b-8e7a-4da0-a3e0-70fad5da5c36
