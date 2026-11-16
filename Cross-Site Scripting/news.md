## Title
Reflected XSS via Hidden Parameter (`title`) in HTML Head - KrazePlanet News

## Description
**Affected URL:** https://news.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet News application via an undocumented, hidden GET parameter (`title`). The application reflects the value of this parameter directly inside the `<title>` HTML tag within the document's `<head>` section without adequate sanitization or output encoding. Because the input is not safely encoded, an attacker can supply a payload that explicitly closes the `<title>` tag (e.g., `</title>`) and injects arbitrary HTML elements. By using a polyglot-style payload with slashes instead of spaces to bypass basic WAF/filters, an attacker can execute arbitrary JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Navigate to the application and observe the standard interface. The visible `fname` and `lname` parameters are not the target of this specific injection.
2. Utilize a parameter discovery tool (e.g., `x8scan`) against the endpoint. The scan will identify an active, hidden parameter named `title`.
3. Inject a payload into the `title` parameter designed to close the `<title>` tag (and other potentially restrictive tags like `<style>` or `<textarea>`) and trigger script execution via an `<img>` tag `onerror` event.
   **Payload:** `1--></Title/</Style/</Textarea/</Iframe/><Img/Src/OnError=(confirm)(1)//`
   *(Note: The payload uses slashes `/` instead of spaces and avoids standard function calls to evade basic keyword and format filters).*
4. Submit the crafted request. The full URL will become:
   `https://news.pwnedbox.site/?title=1--%3E%3C/Title/%3C/Style/%3C/Textarea/%3C/Iframe/%3E%3CImg/Src/OnError=(confirm)(1)//`
5. Observe the page source to confirm the payload successfully broke out of the `<title>` tag, rendering the malicious image tag which immediately executes the `confirm(1)` JavaScript function.

## POC (Proof of Concept)
**Exploit URL:** 
`https://news.pwnedbox.site/?title=1--%3E%3C/Title/%3C/Style/%3C/Textarea/%3C/Iframe/%3E%3CImg/Src/OnError=(confirm)(1)//`

**Visual Evidence:**
* Demonstrates the target application interface.
  <img width="1208" height="577" alt="image" src="https://github.com/user-attachments/assets/d21a8d64-bb81-45b4-86bf-1de468baab22" /><br><br>

* Shows the `x8scan` tool discovering the hidden `title` parameter.
  <img width="840" height="201" alt="image" src="https://github.com/user-attachments/assets/fec3c00b-9cc0-4554-a710-f6d55ff581ac" /><br><br>

* Displays the crafted payload successfully submitted in the URL bar.
  <img width="1125" height="265" alt="image" src="https://github.com/user-attachments/assets/05b55142-c210-4317-b135-64fe27d00e30" /><br><br>

* Confirms the exact reflection within the page source, showing the payload successfully closing the `<title>` tag and breaking into the HTML execution context.
  <img width="962" height="143" alt="image" src="https://github.com/user-attachments/assets/6af53b4e-ed2d-4729-9e0b-c5a6fa25a90c" /><br>


## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately upon page load (since it is injected into the `<head>`). Consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application.
* **Phishing & Redirection:** Altering the page content or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, implement the following actions:
1. **Context-Aware Output Encoding:** Ensure that any user-controllable data reflected between HTML tags (like inside the `<title>` tag) is strictly HTML entity encoded. Characters such as `<`, `>`, `&`, `"`, and `'` must be converted to their corresponding entities (e.g., `<` to `&lt;` and `>` to `&gt;`) so the browser treats them as text rather than executable markup.
2. **Remove Unused Parameters:** If the `title` parameter is a legacy artifact or used only for internal debugging, remove it from the production codebase entirely.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, including inline event handlers like `onerror`.
