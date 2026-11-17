## Title
Reflected XSS via Filter Evasion - KrazePlanet Knowledge Base Search

## Description
**Affected URL:** https://kb.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Knowledge Base application. The application does not properly sanitize or HTML-encode user-supplied input submitted via the `search` GET parameter before reflecting it back to the user on the web page. Furthermore, an attacker can bypass basic keyword-matching security filters (such as a WAF) by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Inject an XSS payload designed to break out of the current HTML attribute into the search input field, or append it directly to the URL parameter. This payload utilizes mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</ScrIPT>`
2. Submit the request. The full URL will become:
   `https://kb.pwnedbox.site/?search=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FScrIPT%3E`
3. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the filter bypass techniques were successful.

## POC (Proof of Concept)
**Exploit URL:** 
`https://kb.pwnedbox.site/?search=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FScrIPT%3E`

**Visual Evidence:**
Refer to the screenshots. They demonstrate the payload successfully rendering within the application's search bar UI and the corresponding JavaScript alert execution resulting from the vulnerable `search` parameter.

<img width="1024" height="591" alt="image" src="https://github.com/user-attachments/assets/025eabfc-f90b-4def-9253-084fe632f20c" /><br>
<img width="1024" height="689" alt="image" src="https://github.com/user-attachments/assets/0bd6cfdd-14d6-492b-8a5f-c35abc18a8c8" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute (like a `value` attribute), strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `search` parameter on the server side using a strict allow-list approach. Avoid relying on block-lists (like blocking the word "alert" or specific script tags), as these are easily bypassed using Unicode encoding and mixed-casing techniques.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
