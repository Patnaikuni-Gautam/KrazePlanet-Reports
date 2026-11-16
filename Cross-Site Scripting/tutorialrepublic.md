## Title
Reflected XSS - TutorialRepublic: Web Development Reference Search

## Description
**Affected URL:** https://tutorialrepublic.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the TutorialRepublic web application. The application fails to properly sanitize or HTML-encode user-supplied input submitted via the `q` URL parameter before reflecting it back to the user on the web page. This allows an attacker to execute arbitrary malicious JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target search endpoint.
2. Inject a standard XSS payload into the search bar, or append it directly to the `q` parameter in the URL. 
   **Payload:** `<script>alert(1)</script>`
3. Submit the request. The full URL will become:
   `https://tutorialrepublic.pwnedbox.site/?q=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized.

## POC (Proof of Concept)
**Exploit URL:** 
`https://tutorialrepublic.pwnedbox.site/?q=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`

**Visual Evidence:**
Refer to the screenshots. It demonstrates the payload successfully rendering within the application's search bar UI and the corresponding URL parameters.

<img width="1820" height="621" alt="image" src="https://github.com/user-attachments/assets/bcddc265-bba2-4509-b827-0a22d9ade58b" />

<img width="1392" height="727" alt="image" src="https://github.com/user-attachments/assets/2a232b4c-8c29-4bf2-90aa-d8e3361825a5" />



## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Impact includes the following:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. For HTML contexts, convert special characters into their corresponding HTML entities (e.g., `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`).
2. **Input Sanitization and Validation:** Validate the `q` parameter on the server side using a strict allow-list approach, ensuring it only accepts expected characters (like alphanumeric characters and safe punctuation) and dropping or sanitizing anything else.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
