## Title
Reflected XSS via Hidden Parameter (`returnTo`) - Shopify Help Center Portal

## Description
**Affected URL:** https://go.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the Shopify Help Center portal (hosted on the pwnedbox.site domain) via an undocumented, hidden GET parameter (`returnTo`). While the application successfully sanitizes the visible input fields on the account confirmation page, it fails to sanitize or encode user input passed to the hidden `returnTo` parameter before reflecting it in the HTTP response. This flaw allows an attacker to break out of the HTML context and inject arbitrary JavaScript that executes within the victim's browser session.

## Steps to Reproduce
1. Navigate to the application and test the exposed visible parameters for XSS. Observe that the application properly encodes these standard inputs, rendering standard payloads inert.
2. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint to identify undocumented parameters. The scan reveals an active, hidden parameter named `returnTo`.
3. Inject a standard HTML/XSS payload into the newly discovered `returnTo` parameter designed to break the current context. 
   **Payload:** `gt'"><script>alert(1)</script>`
4. Submit the crafted request. The full URL will become:
   `https://go.pwnedbox.site/?returnTo=gt%27%22%3C%3E%3Cscript%3Ealert(1)%3C/script%3E`
5. Observe that the payload successfully breaks out of the HTML context and the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://go.pwnedbox.site/?returnTo=gt%27%22%3C%3E%3Cscript%3Ealert(1)%3C/script%3E`

**Visual Evidence:**
* Demonstrates the target account confirmation page where visible fields are present.
<img width="1340" height="895" alt="image" src="https://github.com/user-attachments/assets/407e9233-cef2-4f3b-bf5c-e6751b721fc3" /> <br><br>


* Shows the `x8scan` discovery of the hidden `returnTo` parameter.
<img width="600" height="197" alt="image" src="https://github.com/user-attachments/assets/ce5384ff-08e0-45d0-8405-b0263f750706" /> <br><br>


* Confirms execution of the JavaScript alert payload upon loading the crafted URL.
<img width="1142" height="610" alt="image" src="https://github.com/user-attachments/assets/c413296c-7649-49a5-aa3f-d16208c48de0" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Apply Uniform Sanitization:** Ensure the hidden `returnTo` parameter receives the exact same rigorous context-aware output encoding that is applied to the visible form fields. 
2. **Validate Redirect Parameters:** If `returnTo` is intended for URL redirection, validate the input strictly to ensure it only accepts relative paths or absolute URLs matching a whitelist of trusted domains, preventing both XSS and Open Redirect vulnerabilities.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts.
