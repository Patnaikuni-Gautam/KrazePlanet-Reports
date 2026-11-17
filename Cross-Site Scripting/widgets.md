## Title
Reflected XSS via Hidden Parameter (`post_id`) with Filter Evasion - KrazePlanet Widgets

## Description
**Affected URL:** https://widgets.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Widgets application via an undocumented, hidden GET parameter (`post_id`). The application fails to sanitize or properly HTML-encode user input passed to this hidden parameter before reflecting it in the HTTP response. Furthermore, an attacker can bypass basic keyword-matching security filters (such as a WAF) by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert` instead of `alert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan successfully reveals an active hidden parameter named `post_id`.
2. Inject an XSS payload designed to break out of an HTML attribute into the newly discovered `post_id` parameter. This payload utilizes mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</ScrIPT>`
3. Submit the crafted request. The full URL will become:
   `https://widgets.pwnedbox.site/?post_id=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`
4. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://widgets.pwnedbox.site/?post_id=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`

**Visual Evidence:**
* Shows the `x8scan` discovery of the hidden `post_id` parameter.
  <img width="847" height="195" alt="image" src="https://github.com/user-attachments/assets/3680739a-b9ac-4f45-895c-7ad8ac89e337" /><br><br>

* Confirms execution of the JavaScript alert payload upon loading the crafted URL, demonstrating successful filter evasion and injection.
  <img width="1135" height="732" alt="image" src="https://github.com/user-attachments/assets/52a5811d-ff95-437b-92a0-e2c4b60dee6b" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Consequences include the following:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `post_id` parameter receives rigorous context-aware output encoding. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`).
2. **Remove Unused Parameters:** If the `post_id` parameter is a legacy artifact, testing parameter, or unused in production, remove it entirely to reduce the attack surface.
3. **Robust Input Validation:** Rely on strict server-side allow-listing rather than block-lists, as block-lists are easily bypassed using Unicode encoding and mixed-casing techniques. Validate that `post_id` only accepts expected formats (e.g., integers or UUIDs).
Use Control + Shift + m to toggle the tab key moving focus. Alternatively, use esc then tab to move to the next interactive element on the page.
Attach files by dragging & dropping, selecting or pasting them.
