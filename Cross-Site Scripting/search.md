## Title
Reflected XSS - KrazePlanetLabs GET Parameter Reflection

## Description
**Affected URL:** https://search.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanetLabs search application. The application fails to properly sanitize or HTML-encode user-supplied input submitted via the `fname` (First Name) and `lname` (Last Name) GET parameters before reflecting them back to the user on the web page. This vulnerability allows an attacker to break out of the existing HTML context and execute arbitrary malicious JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute or tag into the `fname` and `lname` input fields, or append them directly to the URL.
   **Payload (fname):** `gt'"><script>alert(3)</script>`
   **Payload (lname):** `gt''><script>alert(3)</script>`
3. Submit the request. The full URL will become:
   `https://search.pwnedbox.site/?fname=gt%27%22%3E%3Cscript%3Ealert%283%29%3C%2Fscript%3E&lname=gt%27%27%3E%3Cscript%3Ealert%283%29%3C%2Fscript%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the HTML context was successfully broken.

## POC (Proof of Concept)
**Exploit URL:** 
`https://search.pwnedbox.site/?fname=gt%27%22%3E%3Cscript%3Ealert%283%29%3C%2Fscript%3E&lname=gt%27%27%3E%3Cscript%3Ealert%283%29%3C%2Fscript%3E`

**Visual Evidence:**
Refer to the below screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript alert execution.

<img width="1170" height="592" alt="image" src="https://github.com/user-attachments/assets/b096bf30-3624-411d-a994-0da8d7d097a2" />

<img width="1207" height="632" alt="image" src="https://github.com/user-attachments/assets/4cb91e39-7de5-4f0d-bd2d-73c08367cb85" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute (like a `value` attribute in an `input` tag), attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `fname` and `lname` parameters on the server side using a strict allow-list approach, ensuring they only accept expected characters (like alphabetical characters for names) and dropping or sanitizing anything else.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
