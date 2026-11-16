## Title
Reflected XSS - PUBG Battlegrounds Community Hub Filter

## Description
**Affected URL:** https://assets.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the filter functionality of the PUBG Battlegrounds Community Hub application. The application does not properly sanitize or HTML-encode user-supplied input submitted via the `p` GET parameter before reflecting it back to the user within the HTML response. This vulnerability allows an attacker to break out of the existing HTML attribute context and inject malicious HTML tags (such as an image tag with an error handler) to execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute and trigger JavaScript execution via an event handler. In this case, an image tag with an invalid source (`src=a`) is used to force the `onerror` event to fire.
   **Payload:** `gt'"><img src=a onerror=alert(1)>`
3. Submit the request through the "FILTER" search bar, or append it directly to the URL. The full URL will become:
   `https://assets.pwnedbox.site/?p=gt%27%22%3C%3E%3Cimg+src%3Da+onerror%3Dalert%281%29%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the HTML context was successfully broken.

## POC (Proof of Concept)
**Exploit URL:** 
`https://assets.pwnedbox.site/?p=gt%27%22%3C%3E%3Cimg+src%3Da+onerror%3Dalert%281%29%3E`

**Visual Evidence:**
Please refer to the screenshot. They demonstrate the payload successfully rendering within the application's filter input field and the corresponding JavaScript alert executing due to the injected `<img>` tag's `onerror` handler.

<img width="1522" height="392" alt="image" src="https://github.com/user-attachments/assets/9a30998d-9f46-406c-a3e1-64396be03d78" />

<img width="952" height="664" alt="image" src="https://github.com/user-attachments/assets/ed887e75-34c3-4459-81c7-c8619f599fe7" />

## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute (like a `value` attribute), attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `p` parameter on the server side using a strict allow-list approach, ensuring it only accepts expected characters (like alphanumeric characters for search filtering) and dropping or sanitizing anything else.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts and event handlers (like `onerror`), which would neutralize this payload even if the injection flaw remains.
