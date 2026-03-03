Reporter: Ahmad Marzouk Security Researcher
Product: WordPress
Vendor: Automattic
Affected Version: 6.9.1
Component: Posts → Add Post > Title Parameter > inject payload >  Preview
Path:

/wp-admin/edit.php?post_type=post
WordPressivpvvslvgp/1c/
Trigger: Clicking Preview
Impact: Stored XSS → Arbitrary JavaScript Execution
Severity: High
CWE: CWE-79 (Improper Neutralization of Input During Web Page Generation)
Attack Type: Authenticated Remote (Admin/Author)
Blast Radius: Any user/editor/admin viewing the previewed post

1. Executive Summary
A stored Cross-Site Scripting (XSS) vulnerability exists in WordPress 6.9.1 within the Posts > Post Preview functionality.
When an authenticated user creates or edits a post, malicious HTML/JavaScript can be injected into the post content.
This payload is then:

Stored permanently in the WordPress database

Executed automatically when editor clicks Preview

This results in full JavaScript execution in the victim’s browser, enabling:

Session hijacking

Admin account takeover

Forced administrative actions

Arbitrary data exfiltration

The issue occurs because WordPress does not sanitize or escape certain HTML event attributes when rendering post previews.

2. Affected Path
The issue is reproducible on the following page:

https://demos2.softaculous.com/WordPressjh9pzqd56n/wp-admin/edit.php?post_type=post
Preview URL example:

/?post_type=post&p=<post_id>&preview=true
Opening this preview renders the injected payload, causing immediate script execution.

3. Vulnerable Parameter
Title
This is stored unsanitized and later injected into the preview page HTML.

4. Steps to Reproduce (Safe)
Step 1 — Log in
Log in as a user with any content-creation rights (Admin, Editor, Author).

Step 2 — Create or Edit a Post
Go to:

Dashboard → Posts → Add New Post
Step 3 — Insert an HTML element with an event attribute
Example harmless test:

<details open ontoggle="alert('XSS Test')">
(WordPress does not sanitize the ontoggle event.)

Step 4 — Click “Preview”
https://demos2.softaculous.com/WordPressjh9pzqd56n/?post_type=post&p=<ID>&preview=true
As soon as the Preview loads, the JavaScript executes, confirming:

It is stored

It affects any viewer

5. Technical Root Cause
The vulnerability arises because:

 User-provided post content is stored without sanitizing inline event attributes
WordPress removes <script> tags, but does not filter dangerous HTML5 event attributes such as:

onload

ontoggle

onclick

onerror

Post Preview renders raw HTML into the DOM
This HTML is inserted directly into the page:

innerHTML → unsafe
 No Content Security Policy (CSP)
This allows:

Inline events

Arbitrary JavaScript execution

DOM manipulation

6. Impact Assessment
Full Admin Account Takeover
Any malicious user with post-edit permission can compromise another admin when they preview the post.

Forced Actions

Using browser-side scripting, an attacker can:

Add new administrators

Install plugins

Change settings

Modify posts

Delete content

 Data Exfiltration

DOM-accessible data and API tokens may be stolen.

Multi-User Environments at Risk
Especially dangerous in:

Multi-author websites

Enterprise WordPress setups

Hosted demo environments

Agency-managed client sites

7. Recommended Fixes for Security Team
1. Sanitize Input (Primary Fix)
Strip or neutralize all inline event attributes:

on*
2. Escape Output
Before rendering post content in preview mode, escape:

<

>

"

'

3. Apply a Strong Content-Security-Policy
Minimum recommended:

default-src 'self';
script-src 'self' 'nonce-<random>';
object-src 'none';
base-uri 'none';
4. Block Unsafe HTML Tags
Ban or sanitize:


<details>

<svg>

<math>

<iframe>

All HTML with event handlers


This PoC Video 
