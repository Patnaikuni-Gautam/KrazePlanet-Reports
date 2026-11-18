## Title
Reflected XSS - KrazePlanet Forms Search Reflection

## Description
**Affected URL:** https://forms.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Forms application's search functionality. The application does not properly sanitize or HTML entity encode user-supplied input submitted via the search field before reflecting it back to the user on the "no results found" page. This flaw allows an attacker to inject arbitrary HTML and JavaScript directly into the document object model (DOM), which executes immediately in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target search endpoint at `https://forms.pwnedbox.site/`.
2. Inject a standard XSS payload into the main search input field.
   **Payload:** `"gt'"<><script>alert(1)</script>`
3. Submit the search query by pressing Enter or clicking the "Search" button.
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized as raw HTML content rather than plain text.

## POC (Proof of Concept)
**Visual Evidence:**
* Demonstrates the XSS payload being entered into the application's search bar.
  <img width="1301" height="847" alt="image" src="https://github.com/user-attachments/assets/c872ff7b-6d44-4fe0-8210-f2a83eeb7f26" /><br><br>

* Confirms the execution of the JavaScript alert payload upon submitting the search.
  <img width="1241" height="912" alt="image" src="https://github.com/user-attachments/assets/24f2b03b-908f-4ee4-85a9-4a9292f7d730" /><br><br>

* Displays the exact reflection within the page source, showing the payload successfully reflected directly inside a `<strong>` tag within the "empty-state" div block without any HTML entity encoding.
<img width="992" height="120" alt="image" src="https://github.com/user-attachments/assets/ac563e8c-0887-4474-9ec8-ccfb62ecd6a1" />


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link containing the payload in the URL parameters. If the victim clicks the link, the attacker's script executes within the victim's session. Realistic consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely HTML entity encoded before being rendered. Characters such as `<`, `>`, `&`, `"`, and `'` must be converted to their corresponding entities (e.g., `<` to `&lt;` and `>` to `&gt;`) so the browser treats the search term strictly as text rather than executable markup.
2. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the output encoding flaw is present.
