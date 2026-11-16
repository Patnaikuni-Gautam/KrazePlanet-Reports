## Title
Reflected XSS - KrazePlanet Profile GET Parameter Reflection

## Description
**Affected URL:** https://profile.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Profile application. The application does not properly sanitize or encode user-supplied input submitted via the `fname` (First Name) and `lname` (Last Name) GET parameters before reflecting them back to the user within the HTML response. This vulnerability allows an attacker to inject payloads that break out of the HTML attribute context and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute into the `fname` and `lname` input fields, or append them directly to the URL parameters. In this instance, a `prompt` function is used instead of an `alert`.
   **Payload (fname & lname):** `gt'"><script>prompt(1)</script>`
3. Submit the request. The full URL will become:
   `https://profile.pwnedbox.site/?fname=gt%27%22%3E%3Cscript%3Eprompt(1)%3C%2Fscript%3E&lname=gt%27%22%3E%3Cscript%3Eprompt(1)%3C%2Fscript%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the HTML context was successfully broken.

## POC (Proof of Concept)
**Exploit URL:** 
`https://profile.pwnedbox.site/?fname=gt%27%22%3E%3Cscript%3Eprompt(1)%3C%2Fscript%3E&lname=gt%27%22%3E%3Cscript%3Eprompt(1)%3C%2Fscript%3E`

**Visual Evidence:**
Rrefer to the following screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript prompt execution.

<img width="1197" height="590" alt="image" src="https://github.com/user-attachments/assets/32cb5ff9-9d6a-4c21-958d-1920e23625bc" />

<img width="1180" height="654" alt="image" src="https://github.com/user-attachments/assets/cfd0ae99-e4dc-4181-b974-ad2e199f3e96" />



## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Includes the following impact:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute (like a `value` attribute), attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `lname` parameter on the server side using a strict allow-list approach, ensuring they only accept expected characters (like alphabetical characters for names) and dropping or sanitizing anything else.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
