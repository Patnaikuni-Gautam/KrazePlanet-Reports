## Title
Reflected XSS via Hidden Parameter (`cat`) with Filter Evasion - KrazePlanet Portal

## Description
**Affected URL:** https://portal.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Portal application via an undocumented, hidden GET parameter (`cat`). While the application successfully sanitizes visible input fields on the interface (like `fname` and `lname`), it fails to sanitize or properly encode user input passed to the hidden `cat` parameter before reflecting it in the HTTP response. Furthermore, an attacker can bypass basic keyword-matching security filters by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan reveals multiple hidden parameters, specifically noting `cat`.
2. Inject an XSS payload designed to break out of an HTML attribute into the `cat` parameter. This payload utilizes mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</ScrIPT>`
3. Submit the crafted request. The full URL will become:
   `https://portal.pwnedbox.site/?cat=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`
4. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://portal.pwnedbox.site/?cat=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`

**Visual Evidence:**
* Shows the `x8scan` discovery of hidden parameters on the target application, including `cat`.
  <img width="1466" height="210" alt="image" src="https://github.com/user-attachments/assets/ea58f7f6-d48a-44fc-b011-54d77967f3c5" /><br><br>

* Confirms execution of the JavaScript alert payload upon loading the crafted URL, demonstrating successful filter evasion and injection.
  <img width="1005" height="582" alt="image" src="https://github.com/user-attachments/assets/68f241b5-e7b3-4cf5-99fb-21870fb0baf2" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Realistic consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `cat` parameter receives rigorous context-aware output encoding. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`).
2. **Remove Unused Parameters:** Review the parameters discovered by the scan (e.g., `cat`, `id`, `number`, `page`). If they are legacy artifacts, testing parameters, or unused in production, remove them entirely to reduce the attack surface.
3. **Robust Input Validation:** Rely on strict server-side allow-listing rather than block-lists, as block-lists are easily bypassed using Unicode encoding and mixed-casing techniques. 
