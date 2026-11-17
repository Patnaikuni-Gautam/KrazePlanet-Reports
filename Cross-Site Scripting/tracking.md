## Title
Reflected XSS via JavaScript Context Breakout - KrazePlanet Tracking

## Description
**Affected URL:** https://tracking.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Tracking search functionality. Unlike standard HTML reflection, the application takes the user-supplied `search` GET parameter and reflects it directly inside an inline JavaScript block on the page. **The vulnerability occurs because the input is reflected without JavaScript-specific encoding inside a string literal, allowing an attacker to inject a double quote (`"`) to prematurely close the string and append executable code.** This allows an attacker to execute arbitrary malicious JavaScript in the context of the victim's browser session.

## Steps to Reproduce
1. Inject an XSS payload designed to break out of a JavaScript string literal. 
   **Payload:** `ibro"*alert(1)*"`
2. Submit the search request. The full URL will become:
   `https://tracking.pwnedbox.site/?search=ibro%22*alert%281%29*%22`
3. **Mechanism Analysis:** When the page loads, the backend renders the payload directly into the `Analytics.trackEvent` function like this: `internalSearchTerm: "ibro"*alert(1)*""`. The injected double quote closes the intended string, while the asterisks (`*`) act as JavaScript multiplication operators. This maintains valid syntax for the JS engine, forcing it to evaluate and execute the `alert(1)` function during the multiplication attempt.
4. Observe that the injected JavaScript executes immediately upon the page loading.

## POC (Proof of Concept)
**Exploit URL:** 
`https://tracking.pwnedbox.site/?search=ibro%22*alert%281%29*%22`

**Visual Evidence:**
* Demonstrates the payload entered into the application's search bar.
  <img width="1482" height="472" alt="image" src="https://github.com/user-attachments/assets/6b435cd8-6519-4948-b651-3fa95062f3ea" /><br><br>

* Displays the exact reflection within the page source, highlighting the vulnerable JavaScript block and the successful string breakout.
  <img width="1006" height="322" alt="image" src="https://github.com/user-attachments/assets/ee8d1fd0-6c3a-449d-8d08-bdc341ebe581" /><br><br>

* Confirms the execution of the JavaScript alert payload upon loading the crafted URL.
  <img width="1352" height="683" alt="image" src="https://github.com/user-attachments/assets/fff13df6-4d6f-4394-acc2-7d8317362953" /><br>

## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes immediately within their session. Realistic consequences include:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **JavaScript Context Encoding:** Standard HTML entity encoding is insufficient when data is reflected inside a `<script>` block. The input must be specifically encoded for a JavaScript context (e.g., escaping quotes `\"`, backslashes `\\`, and control characters).
2. **Safe Data Serialization (Recommended):** The safest way to pass dynamic data from the backend to inline JavaScript is to serialize the data into a JSON string on the server using a secure JSON serialization library (e.g., `json_encode` in PHP), and then safely assign it to a JavaScript variable.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header that bans `unsafe-inline` scripts. This forces all JavaScript to be loaded from external, trusted files, completely eliminating this class of inline injection vulnerabilities.
