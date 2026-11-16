## Title
Reflected XSS - Guidelines for Indian Government Websites and Apps

## Description
**Affected URL:** https://guidelines.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the Guidelines for Indian Government Websites and Apps portal. The application does not properly sanitize or encode user input passed through the `s` (search) URL parameter before reflecting it back onto the page. This flaw allows an attacker to inject and execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target web application.
2. Inject a standard XSS payload into the search bar, or append it directly to the `s` parameter in the URL.
   **Payload:** `<script>alert(1)</script>`
3. Submit the search request. The URL will become:
   `https://guidelines.pwnedbox.site/index.php?s=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`
4. Observe that the injected JavaScript payload executes immediately when the page loads, confirming that the input is reflected without sanitization.

## POC (Proof of Concept)
**Exploit URL:**
`https://guidelines.pwnedbox.site/index.php?s=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`

**Visual Evidence:**
Refer to the below screenshots. They demonstrate the payload successfully rendering within the application's search bar UI and the resulting JavaScript alert execution.

<img width="1337" height="497" alt="image" src="https://github.com/user-attachments/assets/fcd851d3-ea93-429c-bd25-3c33833edcbc" />

<img width="1360" height="660" alt="image" src="https://github.com/user-attachments/assets/e4504259-906a-4521-a88b-6fa3065eca59" />



## Impact
Since this is a Reflected XSS vulnerability, an attacker would need to trick a user into clicking a specially crafted malicious link. If the victim clicks the link, the attacker's script runs within their session. Potential consequences include:
* **Session Hijacking:** Stealing session cookies to impersonate the user and take over their account.
* **Unauthorized Actions:** Performing actions on the application on behalf of the user without their consent.
* **Phishing & Defacement:** Modifying the page content to display fake login forms and steal credentials, or redirecting the user to a malicious website.

## Remediation
To mitigate this vulnerability, implement the following secure coding practices:
1. **Context-Aware Output Encoding:** Ensure that all user-supplied data reflected in HTTP responses is safely encoded for the HTML context. Special characters (such as `<`, `>`, `"`, `'`, and `&`) should be converted into their corresponding HTML entities (e.g., `<` to `&lt;`).
2. **Input Sanitization and Validation:** Validate the `s` parameter on the server side to ensure it only accepts expected characters, stripping out or rejecting any potentially dangerous HTML or JavaScript tags.
3. **Content Security Policy (CSP):** Implement a strict CSP header to restrict the sources from which scripts can be loaded and to prevent the execution of inline scripts.
