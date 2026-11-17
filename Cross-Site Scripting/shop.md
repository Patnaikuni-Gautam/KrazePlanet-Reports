## Title
Reflected XSS via `category` Parameter with Filter Evasion - KrazePlanet Active Experiments

## Description
**Affected URL:** https://shop.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Active Experiments (Shop) application. The application does not properly sanitize or HTML-encode user-supplied input submitted via the `category` GET parameter before reflecting it back to the user on the web page. Furthermore, an attacker can bypass basic keyword-matching security filters (such as a WAF) by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert` instead of `alert`). This allows the attacker to break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint. Observe how the application uses the `category` parameter to filter the displayed labs (e.g., `?category=physics`).
2. Inject an XSS payload designed to break out of the current HTML attribute and bypass basic keyword filter mechanisms.
   **Payload:** `gt'"><ScrIPT>\u0061lert(1)</ScrIPT>`
3. Append the payload to the `category` parameter in the URL. The full URL will become:
   `https://shop.pwnedbox.site/?category=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`
4. Submit the request and observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the filter bypass techniques were successful.

## POC (Proof of Concept)
**Exploit URL:** 
`https://shop.pwnedbox.site/?category=gt%27%22%3C%3E%3CScrIPT%3E\u0061lert(1)%3C/ScrIPT%3E`

**Visual Evidence:**
Refer to the screenshots. They demonstrate the application's default state, the intended use of the `category` parameter for filtering, and the successful execution of the JavaScript alert resulting from the injected payload.

<img width="1023" height="376" alt="image" src="https://github.com/user-attachments/assets/e5ddd01a-9035-4401-b307-dfe1639e4f10" /><br><br>
<img width="1907" height="865" alt="image" src="https://github.com/user-attachments/assets/694c3f43-2e2f-4c9f-ac58-c69cab9ba8d2" /><br><br>
<img width="1361" height="862" alt="image" src="https://github.com/user-attachments/assets/e939ca40-9833-4605-8854-a970ec33558d" /><br>

## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `category` parameter on the server side using a strict allow-list approach, ensuring it only accepts expected category names (like "physics", "chemistry", etc.). Avoid relying on block-lists (like blocking specific script tags or the word "alert"), as these are easily bypassed using Unicode encoding and mixed-casing techniques.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
