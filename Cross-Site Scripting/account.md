## Title
Reflected XSS via WAF/Filter Evasion - KrazePlanet Account

## Description
**Affected URL:** https://account.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Account application. While the application appears to sanitize the `fname` parameter, it fails to properly sanitize or HTML-encode user-supplied input submitted via the `lname` (Last Name) GET parameter before reflecting it back to the user. This vulnerability allows an attacker to break out of the HTML attribute context. Furthermore, the attacker can bypass basic keyword-matching security filters (like a WAF) by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert` instead of `alert`) to execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute into the `lname` input field (the `fname` field can also be injected, but only `lname` reflects unsanitized). This payload uses mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload (lname):** `gt'"><ScrIPT>\u0061lert(1)</SCRIpt>`
3. Submit the request. The full URL will become:
   `https://account.pwnedbox.site/?fname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E&lname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the `lname` input is reflected unsanitized and the filter bypass techniques were successful.

## POC (Proof of Concept)
**Exploit URL:** 
`https://account.pwnedbox.site/?fname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E&lname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E`

**Visual Evidence:**
Refer to the following screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript alert execution resulting from the vulnerable `lname` parameter.

<img width="1202" height="592" alt="image" src="https://github.com/user-attachments/assets/d8b987fa-8837-4774-9c8d-7902ff61668f" /> <br><br>

<img width="1171" height="631" alt="image" src="https://github.com/user-attachments/assets/d6d9837a-eeab-4490-8dc9-1f2b81b76caa" />

## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Consistent Context-Aware Output Encoding:** Ensure that the `lname` parameter receives the exact same rigorous context-aware output encoding that is successfully applied to the `fname` parameter. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`).
2. **Input Sanitization and Validation:** Validate the `lname` parameter on the server side using a strict allow-list approach, ensuring it only accepts expected characters (like alphabetical characters for names). Do not rely on block-lists (like blocking the word "alert" or "<script>"), as these are easily bypassed using Unicode encoding and mixed-casing.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
