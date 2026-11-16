## Title
Reflected XSS - KrazePlanetLabs Feedback GET Parameter Reflection

## Description
**Affected URL:** https://feedback.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanetLabs feedback application. The application does not properly sanitize or encode user-supplied input submitted via the `fname` (First Name) and `lname` (Last Name) GET parameters before reflecting them back to the user within the HTML response. This vulnerability allows an attacker to inject and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute into the `fname` and `lname` input fields, or append them directly to the URL parameters. Note that this payload uses mixed casing (`ScrIPT`) to bypass basic filter rules.
   **Payload (fname & lname):** `gt''"><><ScrIPT>alert(1)</SCRIpt>`
3. Submit the request. The full URL will become:
   `https://feedback.pwnedbox.site/?fname=gt%27%27%22%3C%3E%3CScrIPT%3Ealert%281%29%3C%2FSCRIpt%3E&lname=gt%27%27%22%3C%3E%3CScrIPT%3Ealert%281%29%3C%2FSCRIpt%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and any basic keyword filtering was successfully bypassed.

## POC (Proof of Concept)
**Exploit URL:** 
`https://feedback.pwnedbox.site/?fname=gt%27%27%22%3C%3E%3CScrIPT%3Ealert%281%29%3C%2FSCRIpt%3E&lname=gt%27%27%22%3C%3E%3CScrIPT%3Ealert%281%29%3C%2FSCRIpt%3E`

**Visual Evidence:**
Refer to the screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript alert execution.

<img width="1427" height="572" alt="image" src="https://github.com/user-attachments/assets/85ed7251-bea0-444b-b699-cd0dac5636c9" />

<img width="1191" height="616" alt="image" src="https://github.com/user-attachments/assets/c91474bc-7ba3-492b-860b-ced949f53f27" />



## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include the following:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. For input reflected inside an HTML attribute, attribute encoding must be used to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Rely on strict, server-side allow-listing for input validation rather than block-listing (which was bypassed here using mixed-case tags). Ensure the parameters only accept expected characters (like alphabetical characters for names).
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
