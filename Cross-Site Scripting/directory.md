## Title
Reflected XSS - KrazePlanet Directory GET Parameter Reflection

## Description
**Affected URL:** https://directory.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Directory application. The application fails to properly sanitize or HTML-encode user-supplied input submitted via the `fname` (First Name) and `lname` (Last Name) GET parameters before reflecting them back to the user in the HTML response. This vulnerability allows an attacker to break out of the HTML attribute context and inject an SVG image tag with a malicious `onload` event handler. The use of mixed casing (`SvG`) in the payload successfully bypasses basic keyword-matching security filters, leading to arbitrary JavaScript execution in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute into the `fname` and `lname` input fields, or append them directly to the URL parameters. This payload uses a mixed-case SVG tag to bypass basic filters and executes a script automatically upon rendering.
   **Payload (fname & lname):** `gt'"><SvG onload=alert(1)>`
3. Submit the request. The full URL will become:
   `https://directory.pwnedbox.site/?fname=gt%27%22%3E%3CSvG+onload%3Dalert%281%29%3E&lname=gt%27%22%3E%3CSvG+onload%3Dalert%281%29%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the HTML context was successfully broken to trigger the `onload` event.

## POC (Proof of Concept)
**Exploit URL:** 
`https://directory.pwnedbox.site/?fname=gt%27%22%3E%3CSvG+onload%3Dalert%281%29%3E&lname=gt%27%22%3E%3CSvG+onload%3Dalert%281%29%3E`

**Visual Evidence:**
Refer to the below screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript alert execution via the injected `SvG` tag.

<img width="1216" height="602" alt="image" src="https://github.com/user-attachments/assets/35b4590f-96de-4064-b40a-aef7b6c87130" />

<img width="1144" height="675" alt="image" src="https://github.com/user-attachments/assets/ed252739-61ee-4b15-a8d6-f42d4cd2d702" />



## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `fname` and `lname` parameters on the server side using a strict allow-list approach, ensuring they only accept expected characters (like alphabetical characters for names). Avoid relying on block-lists, as they can often be bypassed with techniques like mixed-case tags.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts and event handlers (like `onload`), which would neutralize this payload even if the injection flaw remains.
