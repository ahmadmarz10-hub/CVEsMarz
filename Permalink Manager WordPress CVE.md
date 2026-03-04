Stored Cross-Site Scripting (XSS) in Permalink Manager Plugin ≤ 2.5.3.1
Summary

A Stored Cross-Site Scripting (XSS) vulnerability exists in the WordPress plugin Permalink Manager ≤ 2.5.3.1.
The vulnerability occurs due to insufficient sanitization and escaping of user-controlled input in post titles or custom permalinks, which are later rendered in the URI Editor interface within the WordPress admin panel.

An attacker with Contributor or Author privileges can inject malicious JavaScript payloads that will execute when an administrator visits the Permalink Manager URI Editor page.

This may lead to administrator session hijacking, privilege escalation, and full WordPress site compromise.

Vulnerability Details
Affected Component

Plugin: Permalink Manager
Affected Version: ≤ 2.5.3.1

Affected Page:

/wp-admin/tools.php?page=permalink-manager

Feature:

URI Editor
Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

OWASP Category:

A03:2021 – Injection

CWE:

CWE-79 Improper Neutralization of Input During Web Page Generation
Root Cause

User-supplied input (post title, slug, or custom permalink) is stored in the database and later rendered inside the URI Editor interface without proper escaping.

The plugin outputs user-controlled values directly into the HTML page.

Example vulnerable pattern:

echo $post_title;

Secure implementation should use proper escaping:

echo esc_html($post_title);

or

echo esc_attr($post_title);

Because escaping is missing, injected JavaScript executes when the page loads.

Attack Scenario

An attacker obtains Contributor or Author access to the WordPress site.

The attacker creates or edits a post.

The attacker injects a malicious payload in the post title or permalink field.

The payload is saved in the database.

When an administrator opens the Permalink Manager URI Editor, the malicious script executes in the admin's browser.

Proof of Concept (PoC)
Step 1 — Login as Contributor

Login to the WordPress dashboard with Contributor privileges.

Step 2 — Create or Edit a Post

Navigate to:

Posts → Add New
Step 3 — Inject Payload

Insert the following payload in the Post Title:

<details/open/ontoggle=prompt(origin)>

Save or publish the post.

Step 4 — Trigger the Vulnerability

Navigate to:

Tools → Permalink Manager → URI Editor

or directly visit:

/wp-admin/tools.php?page=permalink-manager
Result

The malicious JavaScript payload executes in the administrator's browser.

Impact

This vulnerability allows attackers to execute arbitrary JavaScript within the administrator’s session.

Possible impacts include:

• Admin session hijacking
• Creation of new administrator accounts
• Installation of malicious plugins
• Uploading backdoors
• Complete WordPress site takeover

Because the script executes in the admin context, the impact is high severity.

CVSS v3.1 Score (Estimated)
CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:H/I:H/A:L

Score:

8.8 – High
Recommended Fix

All user-controlled output should be escaped before rendering.

Example fix:

echo esc_html($post_title);

or

echo esc_attr($permalink);

Additionally:

• Sanitize input when saving data
• Escape output when rendering data

Recommended WordPress functions:

sanitize_text_field()
esc_html()
esc_attr()
esc_url()
References

OWASP XSS Prevention:

https://owasp.org/www-community/attacks/xss/

WordPress Escaping Documentation:

https://developer.wordpress.org/apis/security/escaping/
esc_attr()

Ensuring output escaping in the URI Editor interface would prevent malicious HTML from being executed.

https://github.com/user-attachments/assets/41715df8-a94b-4376-841e-484db72407cb
