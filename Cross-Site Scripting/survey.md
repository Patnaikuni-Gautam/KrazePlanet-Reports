## Title
Self XSS via POST Parameter - KrazePlanet Survey Form

## Description
**Affected URL:** https://survey.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Survey application. The application fails to properly sanitize or HTML-encode user-supplied input submitted via the form's POST request before reflecting it back to the user. Specifically, the input is reflected directly inside the `value` attribute of an `<input>` HTML tag. Because the payload is submitted via a POST request rather than a GET URL, this operates primarily as a Self-XSS vulnerability. By injecting double quotes (`"`), an attacker can break out of the HTML attribute context. Furthermore, the use of mixed-case HTML tags (`<ScrIPT>`) successfully bypasses basic keyword-matching security filters, allowing the execution of arbitrary JavaScript within the user's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint (`https://survey.pwnedbox.site/`).
2. In the "What's your name?" input field, inject an XSS payload designed to break out of an HTML attribute and evade basic filters.
   **Payload:** `gt'"><ScrIPT>alert(1)</ScrIPT>`
3. Click the "Enter" button to submit the form via a POST request. 
4. Observe that the injected JavaScript executes immediately upon the response loading, demonstrating that the payload successfully closed the `value` attribute and broke into the executable HTML context.

## POC (Proof of Concept)
**Exploit Mechanism:** 
The payload must be submitted via an HTTP POST request to `https://survey.pwnedbox.site/`.

**Visual Evidence:**
* Demonstrates the XSS payload being entered into the application's form field.
  <img width="1162" height="352" alt="image" src="https://github.com/user-attachments/assets/d64cae09-448e-4161-8c54-7ca0ed72f0a6" /><br><br>

* Confirms the execution of the JavaScript alert payload upon form submission.
  <img width="1132" height="742" alt="image" src="https://github.com/user-attachments/assets/cc1b28e9-f850-4928-b395-06672de55a7c" /><br><br>

* Displays the exact reflection within the page source. It shows the payload successfully breaking out of the `<input value="...">` attribute without HTML entity encoding, turning the injected payload into executable markup.
  <img width="853" height="100" alt="image" src="https://github.com/user-attachments/assets/e8ec4242-d024-47f7-9cf5-387f9d5003f8" />

## Impact
While Self-XSS alone typically has a lower impact (since the attacker is attacking their own session), the security consequences increase significantly if this vulnerability is chained with other flaws:
* **CSRF to XSS:** If the form lacks Cross-Site Request Forgery (CSRF) protection, an attacker can host a malicious web page that silently forces the victim's browser to submit the POST request containing the XSS payload. 
* **Session Hijacking & Phishing:** Once successfully chained and executed in a victim's browser, the attacker can steal session cookies, perform unauthorized actions on the user's behalf, or manipulate the DOM to harvest credentials.

## Remediation
To properly fix this vulnerability, implement the following secure coding practices:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on its location in the DOM. Because the input is reflected inside an HTML attribute, strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from closing the attribute prematurely.
2. **Implement CSRF Protection:** Ensure the form utilizes unpredictable, session-specific Anti-CSRF tokens. This prevents attackers from forcing victims to submit malicious POST requests, heavily mitigating the risk of chaining attacks.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts.
