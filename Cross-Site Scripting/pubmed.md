## Title
Reflected XSS - PubMed Advanced Search Builder

## Description
**Affected URL:** https://pubmed.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the PubMed Advanced Search Builder functionality. The application does not properly sanitize or HTML-encode user-supplied input submitted via the `term` URL parameter before reflecting it back to the user on the web page. This allows an attacker to execute arbitrary malicious JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target search endpoint.
2. Inject a standard XSS payload into the "Add terms to the query box" input field, or append it directly to the `term` parameter in the URL. 
   **Payload:** `<script>alert(2)</script>`
3. Submit the request. The full URL will become:
   `https://pubmed.pwnedbox.site/index.php?term=%3Cscript%3Ealert%282%29%3C%2Fscript%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized.

## POC (Proof of Concept)
**Exploit URL:** 
`https://pubmed.pwnedbox.site/index.php?term=%3Cscript%3Ealert%282%29%3C%2Fscript%3E`

**Visual Evidence:**
Refer to the following screenshots. They demonstrate the payload successfully rendering within the application's search builder UI and the corresponding JavaScript alert execution.

<img width="1360" height="692" alt="image" src="https://github.com/user-attachments/assets/ff1c08ac-c641-44c8-93a5-7622228d8ec6" />


<img width="1326" height="625" alt="image" src="https://github.com/user-attachments/assets/5ef88577-9ac4-4385-b74e-56c6788a1896" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Potential impact include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. For HTML contexts, convert special characters into their corresponding HTML entities (e.g., `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`).
2. **Input Sanitization and Validation:** Validate the `term` parameter on the server side using a strict allow-list approach, ensuring it only accepts expected characters and dropping or sanitizing anything else.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
