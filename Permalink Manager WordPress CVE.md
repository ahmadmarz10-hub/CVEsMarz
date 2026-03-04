Stored Cross-Site Scripting (XSS) in Permalink Manager WordPress Plugin

Product: Permalink Manager (WordPress Plugin)
Vendor: Permalink Manager Developers (Maciej Bis)
Version: 2.5.3.1
Affected Component: URI Editor / Post Title Rendering
Vulnerability Type: Stored Cross-Site Scripting (XSS)
CWE: CWE-79 — Improper Neutralization of Input During Web Page Generation
Severity: High
Attack Type: Authenticated (Admin / Editor)
Impact: Arbitrary JavaScript execution in administrator browser

Summary

A Stored Cross-Site Scripting (XSS) vulnerability was discovered in the Permalink Manager WordPress plugin.
The vulnerability occurs when user-supplied input placed in the post title > Edit > Rename > name paramter > inject the payload > save > back to dashboard > click view > the payload working!! field is not properly sanitized before being stored and later rendered in administrative pages.

An attacker can inject a malicious HTML payload into the post name. The payload is stored in the database and later executed when viewed in certain administrative interfaces such as the Permalink Manager URI Editor.

Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

Proof of Concept (PoC)
Payload
details/open/ontoggle=prompt(origin)
Steps to Reproduce

Log in to the WordPress admin panel.

Navigate to an existing Permalink Manager posttitle editing:

/wp-admin/post.php?post=1&action=edit

Update the post title > rename > name > with the following payload:

<details/open/ontoggle=prompt(origin)>

Save or update the post.

Navigate to the Permalink Manager URI Editor page:

/wp-admin/tools.php?page=permalink-manager&section=uri_editor&orderby=post_title&order=asc
 Click View > the payload is working !!
 
Example test environment:

https://demos2.softaculous.com/WordPress3kq2dw6c6e/wp-admin/tools.php?page=permalink-manager&section=uri_editor&orderby=post_title&order=asc

Locate the modified post entry.

When the stored value is rendered, interacting with the injected < details > element triggers JavaScript execution.

Result

The injected payload executes JavaScript in the administrator's browser:

prompt(origin)

This confirms a Stored XSS vulnerability, as the payload persists in the database and executes whenever the affected page loads or the element is interacted with.

Affected Locations

Payload is rendered in the following admin pages:

/wp-admin/tools.php?page=permalink-manager&section=uri_editor
/wp-admin/post.php?post=1&action=edit
Impact

An attacker with privileges to create or modify posts could exploit this vulnerability to:

Execute arbitrary JavaScript in an administrator's browser

Perform administrative actions via CSRF

Steal authentication cookies or nonces

Inject malicious content into the WordPress dashboard

Because the payload is stored, it will execute whenever the affected page is viewed.

Recommendation

Developers should properly sanitize and escape user input before storing or displaying it.

Suggested protections include:

sanitize_text_field()

wp_kses()

esc_html()

esc_attr()

Ensuring output escaping in the URI Editor interface would prevent malicious HTML from being executed.

https://github.com/user-attachments/assets/41715df8-a94b-4376-841e-484db72407cb
