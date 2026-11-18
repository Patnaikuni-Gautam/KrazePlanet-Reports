## Title
Self XSS via POST Parameter - KrazePlanet Apply Form

## Description
**Affected URL:** https://apply.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Apply application. The application does not properly sanitize or HTML-encode user-supplied input submitted via the form's POST request before reflecting it back to the user within the HTML response. Because the payload is delivered via a POST request rather than a shareable GET URL, this is primarily a Self-XSS vulnerability. An attacker can also bypass basic keyword-matching security filters by utilizing mixed-case HTML tags (`<ScrIPT>`). This allows the execution of arbitrary JavaScript in the context of the user's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint (`https://apply.pwnedbox.site/`).
2. In the "What's your name?" input field, inject an XSS payload. This payload utilizes mixed-case tags to bypass basic filter mechanisms.
   **Payload:** `gt'"><ScrIPT>alert(1)</ScrIPT>`
3. Click the "Enter" button to submit the form via a POST request. 
4. Observe that the payload is reflected unsanitized directly within the `<p>` tags of the page's HTML, and the injected JavaScript executes immediately upon the response loading.

## POC (Proof of Concept)
**Exploit Mechanism:** 
The payload must be submitted via an HTTP POST request to `https://apply.pwnedbox.site/`.

**Visual Evidence:**
* Demonstrates the payload being entered into the application's form field.
  <img width="1087" height="430" alt="image" src="https://github.com/user-attachments/assets/a2a9eca6-0e45-4191-8308-986824b2225c" /><br><br>

* Confirms the execution of the JavaScript alert payload upon form submission.
  <img width="1042" height="706" alt="image" src="https://github.com/user-attachments/assets/fe62866e-4f1b-4391-b9be-acc6bd360521" /><br><br>
  
* Displays the exact reflection within the page source, showing the payload successfully reflected directly inside a `<p class="text-center">` tag without HTML entity encoding.
  <img width="805" height="155" alt="image" src="https://github.com/user-attachments/assets/00927da3-b7c4-4c1a-a685-cd748eecdc69" />


## Impact
On its own, Self-XSS has a low impact because an attacker can typically only execute code within their own session. However, the realistic security consequences increase significantly if this vulnerability is chained with others:
* **CSRF to XSS:** If the form lacks Cross-Site Request Forgery (CSRF) protection, an attacker can create a malicious web page that forces the victim's browser to submit the POST request containing the XSS payload. 
* **Session Hijacking & Phishing:** Once successfully chained and executed in a victim's browser, the attacker can steal session cookies, perform unauthorized actions on the user's behalf, or deface the page to steal credentials.

## Remediation
To properly fix this vulnerability, implement the following secure coding practices:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely HTML entity encoded. Characters such as `<`, `>`, `&`, `"`, and `'` must be converted to their corresponding entities (e.g., `<` to `&lt;` and `>` to `&gt;`) before being rendered inside HTML tags.
2. **Implement CSRF Protection:** Ensure the form utilizes unpredictable, session-specific Anti-CSRF tokens to prevent attackers from forcing victims to submit malicious POST requests, mitigating the risk of chaining.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
