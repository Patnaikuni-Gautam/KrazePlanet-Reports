## Title
Reflected XSS via Hidden Parameter (`username`) with Filter Evasion - KrazePlanet Media Messages

## Description
**Affected URL:** https://media.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Media messaging portal via an undocumented, hidden GET parameter (`username`). The application fails to properly sanitize or encode user input passed to this hidden parameter before reflecting it in the HTTP response. Furthermore, an attacker can bypass basic keyword-matching security filters by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert` instead of `alert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Navigate to the application endpoint. No visible vulnerable input fields are immediately apparent on the interface.
2. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan reveals an active, hidden parameter named `username`.
3. Inject an XSS payload designed to break out of an HTML attribute into the newly discovered `username` parameter. This payload utilizes mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</SCRIpt>`
4. Submit the crafted request. The full URL will become:
   `https://media.pwnedbox.site/?username=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/SCRIpt%3E`
5. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://media.pwnedbox.site/?username=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/SCRIpt%3E`

**Visual Evidence:**
* Demonstrates the target messaging application interface without visible vulnerable parameters.
  <img width="1175" height="686" alt="image" src="https://github.com/user-attachments/assets/cae9c6bc-2471-41f7-8f80-7252696eae0e" /><br><br>

* Shows the `x8scan` discovery of the hidden `username` parameter.
  <img width="627" height="198" alt="image" src="https://github.com/user-attachments/assets/a210a427-c6a5-4700-b200-2896701398bc" /><br><br>

* Confirms execution of the JavaScript alert payload upon loading the crafted URL, demonstrating successful filter evasion.
  <img width="1055" height="613" alt="image" src="https://github.com/user-attachments/assets/e046aded-d35e-4e1d-a958-b73e7a76ad6c" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf (e.g., sending messages or altering profile data).
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `username` parameter receives rigorous context-aware output encoding. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`).
2. **Remove Unused Parameters:** If the `username` parameter is a legacy artifact or used only for debugging, remove it from the production codebase entirely to reduce the attack surface.
3. **Robust Input Validation:** Rely on strict server-side allow-listing rather than block-lists, as they are easily bypassed using Unicode encoding and mixed-casing techniques. 
4. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts.
