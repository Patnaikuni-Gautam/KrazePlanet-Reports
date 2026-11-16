## Title
Reflected XSS - KrazePlanet Support GET Parameter Reflection

## Description
**Affected URL:** https://support.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Support application. The application does not properly sanitize or HTML-encode user-supplied input submitted via the `fname` (First Name) and `lname` (Last Name) GET parameters before reflecting them back to the user in the HTML response. This vulnerability allows an attacker to break out of the HTML attribute context and inject an SVG image tag with a malicious `onload` event handler, leading to arbitrary JavaScript execution in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute into the `fname` and `lname` input fields, or append them directly to the URL parameters. This specific payload utilizes an SVG tag to execute script automatically upon rendering.
   **Payload (fname & lname):** `gt'"><svg onload=alert(1)>`
3. Submit the request. The full URL will become:
   `https://support.pwnedbox.site/?fname=gt%27%22%3E%3Csvg+onload%3Dalert%281%29%3E&lname=gt%27%22%3E%3Csvg+onload%3Dalert%281%29%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the HTML context was successfully broken to trigger the `onload` event.

## POC (Proof of Concept)
**Exploit URL:** 
`https://support.pwnedbox.site/?fname=gt%27%22%3E%3Csvg+onload%3Dalert%281%29%3E&lname=gt%27%22%3E%3Csvg+onload%3Dalert%281%29%3E`

**Visual Evidence:**
Refer to the screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript alert execution via the injected SVG tag.

<img width="1410" height="607" alt="image" src="https://github.com/user-attachments/assets/bd9fd90e-ec82-453c-9ef6-546e1256459d" />

<img width="1177" height="633" alt="image" src="https://github.com/user-attachments/assets/43817d0a-1b28-46a8-ac06-2b8db4917136" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Impact includes:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute (like a `value` attribute), strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `fname` and `lname` parameters on the server side using a strict allow-list approach, ensuring they only accept expected characters (like alphabetical characters for names) and dropping or sanitizing anything else.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts and event handlers (like `onload`), which would neutralize this payload even if the injection flaw remains.
