## Title
Reflected XSS via Hidden Parameter (`cat`) with Filter Evasion - KrazePlanet Checkout

## Description
**Affected URL:** https://checkout.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Checkout application via an undocumented, hidden GET parameter (`cat`). While the application successfully sanitizes visible input fields (`fname` and `lname`), it fails to sanitize or properly encode user input passed to the hidden `cat` parameter before reflecting it in the HTTP response. Furthermore, an attacker can bypass basic keyword-matching security filters by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan reveals multiple hidden parameters, including `cat`.
2. Inject an XSS payload designed to break out of an HTML attribute into the `cat` parameter. This payload utilizes mixed-case tags and Unicode escaping to bypass filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</ScrIPT>`
3. Submit the crafted request. The full URL will become:
   `https://checkout.pwnedbox.site/?cat=gt%27%22%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`
4. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://checkout.pwnedbox.site/?cat=gt%27%22%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`

**Visual Evidence:**
* Demonstrates the target application interface where standard parameters are safe.
  <img width="1082" height="588" alt="image" src="https://github.com/user-attachments/assets/bd0f3bcb-29e3-4d71-8b99-2879895dd9b5" /> <br><br>
* Shows the `x8scan` discovery of hidden parameters, specifically `cat`.
  <img width="1452" height="195" alt="image" src="https://github.com/user-attachments/assets/9b5d1aa1-c9e1-423a-a17c-dddcb90cbeb0" /> <br><br>
* Confirms execution of the JavaScript alert payload upon loading the crafted URL, demonstrating successful filter evasion.
  <img width="1210" height="572" alt="image" src="https://github.com/user-attachments/assets/1524172a-9acf-428b-bab6-25df8cca9947" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `cat` parameter receives rigorous context-aware output encoding. If the input is reflected inside an HTML attribute, strict attribute encoding must be used.
2. **Remove Unused Parameters:** Review the parameters discovered by the scan (e.g., `cat`, `id`, `number`, `page`). If they are legacy artifacts or unused in production, remove them entirely to reduce the attack surface.
3. **Robust Input Validation:** Rely on strict server-side allow-listing rather than block-lists, as they are easily bypassed using Unicode encoding and mixed-casing techniques. 
