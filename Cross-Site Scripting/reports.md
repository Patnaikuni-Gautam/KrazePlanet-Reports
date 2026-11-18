## Title
Self XSS via POST Parameter in HTML Title Tag - KrazePlanet Reports

## Description
**Affected URL:** https://reports.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Reports application. The application does not properly sanitize or HTML entity encode user-supplied input submitted via the "Name" POST parameter before reflecting it directly inside the `<title>` tag of the HTML document's `<head>`. Because the payload is submitted via an HTTP POST request, this is primarily a Self-XSS vulnerability. An attacker can break out of the `<title>` tag context using a polyglot-style payload (utilizing slashes instead of spaces to bypass basic WAFs) to inject a new HTML element that executes arbitrary JavaScript.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint (`https://reports.pwnedbox.site/`).
2. In the "What's your name?" input field, inject a specialized XSS payload. This payload is designed to explicitly close the `<title>` tag (and other restrictive tags) and inject a `<Details>` element with an `OnToggle` event handler that triggers automatically.
   **Payload:** `<!--></Title/</Textarea/</Script/</Iframe/><Details/Open/OnToggle=(confirm)(1)//`
3. Click the "Enter" button to submit the form via a POST request.
4. Observe that the payload successfully breaks out of the `<title>` tag, renders the injected `<Details>` element, and immediately executes the `confirm(1)` JavaScript function.

## POC (Proof of Concept)
**Exploit Mechanism:** 
The payload must be submitted via an HTTP POST request to `https://reports.pwnedbox.site/`.

**Visual Evidence:**
* Demonstrates the polyglot payload being entered into the application's form field.
  <img width="1150" height="417" alt="image" src="https://github.com/user-attachments/assets/a9cce6c9-28e6-449b-8321-9b4ee0e7ecca" /><br><br>

* Confirms the execution of the JavaScript confirmation dialogue upon form submission.
  <img width="1122" height="720" alt="image" src="https://github.com/user-attachments/assets/383a78f1-81af-4a5f-bf88-ca8155e9a5a2" /><br><br>

* Displays the exact reflection within the page source, showing the payload successfully closing the `<title>` tag and breaking into the HTML execution context without encoding.
  <img width="1236" height="62" alt="image" src="https://github.com/user-attachments/assets/d09dda62-fc24-412a-8564-6c5f727f9c87" />


## Impact
On its own, Self-XSS has a low direct impact because an attacker is primarily attacking their own session. However, the security consequences escalate significantly if this vulnerability is chained with other flaws:
* **CSRF to XSS:** If the form lacks Cross-Site Request Forgery (CSRF) protection, an attacker can create a malicious web page that forces the victim's browser to silently submit the POST request containing the XSS payload. 
* **Session Hijacking & Phishing:** Once chained and successfully executed in a victim's browser, the attacker can steal session cookies, perform unauthorized actions on the user's behalf, or redirect the user to a malicious site.

## Remediation
To properly fix this vulnerability, implement the following secure coding practices:
1. **Context-Aware Output Encoding:** Ensure that any user-controllable data reflected between HTML tags (especially inside the `<title>` tag) is strictly HTML entity encoded. Characters such as `<`, `>`, `&`, `"`, and `'` must be converted to their corresponding entities (e.g., `<` to `&lt;` and `>` to `&gt;`) so the browser treats them as text rather than executable markup.
2. **Implement CSRF Protection:** Ensure the form utilizes unpredictable, session-specific Anti-CSRF tokens to prevent attackers from forcing victims to submit malicious POST requests.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts and event handlers (like `OnToggle`), which would neutralize this payload even if the injection flaw remains.
