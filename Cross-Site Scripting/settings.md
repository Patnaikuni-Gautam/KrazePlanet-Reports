## Title
Reflected XSS via Hidden Parameter (`academy`) with Filter Evasion - KrazePlanet Settings

## Description
**Affected URL:** https://settings.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Settings application via an undocumented, hidden GET parameter (`academy`). While the application successfully sanitizes visible input fields on the interface, it fails to sanitize or properly encode user input passed to the hidden `academy` parameter before reflecting it in the HTTP response. Additionally, an attacker can bypass basic keyword-matching security filters by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan reveals a large number of hidden parameters, specifically noting `academy` at the end of the list.
2. Inject an XSS payload designed to break out of an HTML attribute into the `academy` parameter. This payload utilizes mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</ScrIPT>`
3. Submit the crafted request. The full URL will become:
   `https://settings.pwnedbox.site/?academy=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`
4. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://settings.pwnedbox.site/?academy=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`

**Visual Evidence:**
* Shows the `x8scan` discovery of hidden parameters on the target application, including `academy`.
  <img width="1462" height="282" alt="image" src="https://github.com/user-attachments/assets/819b6186-0a30-4c63-a7a6-7a3922484613" /><br><br>

* `image_a54bd9.png`: Confirms execution of the JavaScript alert payload upon loading the crafted URL, demonstrating successful filter evasion and injection.
  <img width="1072" height="657" alt="image" src="https://github.com/user-attachments/assets/1a5621de-bf23-415c-872d-867833dd4dd6" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Realistic consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `academy` parameter receives rigorous context-aware output encoding. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`).
2. **Remove Unused Parameters:** Review the extensive list of parameters discovered by the scan. If they are legacy artifacts, internal testing flags, or otherwise unused in production, remove them entirely to reduce the attack surface.
3. **Robust Input Validation:** Rely on strict server-side allow-listing rather than block-lists, as they are easily bypassed using Unicode encoding and mixed-casing techniques. 
