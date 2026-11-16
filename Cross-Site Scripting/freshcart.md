## Title
Reflected XSS - FreshCart Grocery Delivery

## Description
**Affected URL:** https://freshcart.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the search functionality of the FreshCart web application. The application does not properly sanitize or HTML-encode user-supplied input submitted via the `q` URL parameter before reflecting it back to the user within the page content. This flaw allows an attacker to inject and execute arbitrary malicious JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target web application.
2. Inject a standard XSS payload into the main search bar, or append it directly to the `q` parameter in the URL.
   **Payload:** `<script>alert(3)</script>`
3. Submit the search request. The URL will become:
   `https://freshcart.pwnedbox.site/index.php?q=%3Cscript%3Ealert%283%29%3C%2Fscript%3E`
4. Observe that the injected JavaScript payload executes immediately when the page loads, confirming that the input is reflected without proper sanitization.

## POC (Proof of Concept)
**Exploit URL:**
`https://freshcart.pwnedbox.site/index.php?q=%3Cscript%3Ealert%283%29%3C%2Fscript%3E`

**Visual Evidence:**
Refer to the below screenshots. They demonstrate the payload successfully rendering within the application's search bar UI and the subsequent JavaScript alert execution.

<img width="1457" height="562" alt="image" src="https://github.com/user-attachments/assets/afa5de9f-f80c-487f-b3fb-3774af0c5431" />


<img width="1342" height="605" alt="image" src="https://github.com/user-attachments/assets/63bbbd61-4a48-401a-a4f0-bbba3efb860f" />



## Impact
Because this is a Reflected XSS vulnerability, an attacker would need to socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script runs within their session. Potential consequences include:
* **Session Hijacking:** Stealing session cookies to impersonate the user and take over their account.
* **Unauthorized Actions:** Performing actions on the application on behalf of the user without their consent (e.g., modifying account details or placing orders).
* **Phishing & Defacement:** Modifying the page content to display fake login forms and steal credentials, or redirecting the user to a malicious website.

## Remediation
To mitigate this vulnerability, implement the following secure coding practices:
1. **Context-Aware Output Encoding:** Ensure that all user-supplied data reflected in HTTP responses is safely encoded for the HTML context before being rendered. Special characters (such as `<`, `>`, `"`, `'`, and `&`) should be converted into their corresponding HTML entities.
2. **Input Sanitization and Validation:** Validate the `q` parameter on the server side using a strict allow-list to ensure it only accepts expected characters, stripping out or rejecting any potentially dangerous HTML or JavaScript tags.
3. **Content Security Policy (CSP):** Implement a strict CSP header to restrict the sources from which scripts can be loaded and to prevent the execution of unauthorized inline scripts.
