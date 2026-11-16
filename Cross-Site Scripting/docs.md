## Title
Reflected XSS via Hidden Parameter (`q`) - KrazePlanet Labs

## Description
**Affected URL:** https://docs.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Labs application via an undocumented, hidden GET parameter (`q`). While the application successfully sanitizes the visible input fields (`fname` and `lname`), preventing standard injection attacks, it fails to sanitize or encode user input passed to the hidden `q` parameter. This flaw allows an attacker to bypass filters using mixed-case tags and inject arbitrary JavaScript that executes immediately in the context of the victim's browser session.

## Steps to Reproduce
1. Navigate to the application and test the exposed `fname` and `lname` parameters for XSS. Observe that the application properly encodes these inputs, rendering standard payloads inert.
2. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan reveals an active, hidden parameter named `q`.
3. Inject an HTML/XSS payload into the newly discovered `q` parameter. This payload uses mixed casing (`ScRIpt`) to bypass any basic keyword filters and breaks out of the context to execute a confirmation dialogue.
   **Payload:** `'"<><ScRIpt>confirm(1)</ScRIpt>`
4. Submit the crafted request. The full URL will become:
   `https://docs.pwnedbox.site/?q=%27%22%3C%3E%3CScRIpt%3Econfirm(1)%3C/ScRIpt%3E`
5. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://docs.pwnedbox.site/?q=%27%22%3C%3E%3CScRIpt%3Econfirm(1)%3C/ScRIpt%3E`

**Visual Evidence:**
* Demonstrates proper encoding on standard visible parameters.

<img width="1207" height="582" alt="image" src="https://github.com/user-attachments/assets/2023f631-807b-47a4-8592-f909bcaa01a5" />


* Shows the `x8scan` discovery of the hidden `q` parameter.

<img width="803" height="196" alt="image" src="https://github.com/user-attachments/assets/4d5f4070-dafa-4195-940c-45fb59aef0e3" />

Confirms execution of the JavaScript payload upon interaction
<img width="1205" height="623" alt="image" src="https://github.com/user-attachments/assets/f28f0106-dafc-48db-8c4a-5bfd608abf40" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `q` parameter (if required for application functionality) receives the exact same rigorous context-aware output encoding that is successfully applied to the `fname` and `lname` parameters. Rely on strict server-side allow-listing rather than block-listing.
2. **Remove Unused Parameters:** If the `q` parameter is a legacy artifact or used only for debugging, remove it from the production codebase entirely to reduce the attack surface.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts.
