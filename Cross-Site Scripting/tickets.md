## Title
Reflected XSS via WAF/Filter Evasion - KrazePlanet Tickets GET Parameter Reflection

## Description
**Affected URL:** https://tickets.pwnedbox.site/

A Reflected Cross-Site Scripting (XSS) vulnerability exists within the KrazePlanet Tickets application. The application fails to properly sanitize or HTML-encode user-supplied input submitted via the `fname` (First Name) and `lname` (Last Name) GET parameters before reflecting them back to the user in the HTML response. This vulnerability allows an attacker to break out of the HTML attribute context. Furthermore, the attacker can successfully bypass basic keyword-matching security filters (such as a WAF) by utilizing mixed-case HTML tags (`<ScrIPT>`) and JavaScript Unicode escape sequences (e.g., `\u0061lert` instead of `alert`), leading to arbitrary JavaScript execution in the context of the victim's browser session.

## Steps to Reproduce
1. Open a web browser and navigate to the target endpoint.
2. Inject an XSS payload designed to break out of the current HTML attribute into the `fname` and `lname` input fields, or append them directly to the URL parameters. This payload utilizes mixed-case tags and Unicode escaping to bypass basic filter mechanisms.
   **Payload (fname & lname):** `gt'"><ScrIPT>\u0061lert(1)</SCRIpt>`
3. Submit the request. The full URL will become:
   `https://tickets.pwnedbox.site/?fname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E&lname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E`
4. Observe that the injected JavaScript executes immediately upon the page loading, demonstrating that the input is reflected unsanitized and the filter bypass techniques were successful.

## POC (Proof of Concept)
**Exploit URL:** 
`https://tickets.pwnedbox.site/?fname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E&lname=gt%27%22%3C%3E%3CScrIPT%3E%5Cu0061lert%281%29%3C%2FSCRIpt%3E`

**Visual Evidence:**
Refer to the following screenshots. They demonstrate the payloads successfully rendering within the application's input fields and the corresponding JavaScript alert execution resulting from the vulnerable parameters.

<img width="1372" height="592" alt="image" src="https://github.com/user-attachments/assets/3cae46ee-6bba-4f68-b93a-1d7bd07d425b" />
<img width="1187" height="620" alt="image" src="https://github.com/user-attachments/assets/79bd955a-886c-493a-ab13-84fe64d9de82" />



## Impact
Because this is a Reflected XSS vulnerability, an attacker must socially engineer a victim into clicking a maliciously crafted link. If the victim clicks the link, the attacker's script executes within the victim's session. Consequences include the following:
* **Session Hijacking:** Stealing session cookies or authentication tokens to take over the user's account.
* **Unauthorized Actions:** Forcing the user's browser to perform unintended actions on the application on their behalf.
* **Phishing & Redirection:** Altering the appearance of the page to steal credentials or redirecting the user to a malicious external site.

## Remediation
To properly fix this vulnerability, the following secure coding practices should be implemented:
1. **Context-Aware Output Encoding:** Ensure that all user-controllable data returned in HTTP responses is safely encoded based on where it is placed in the HTML document. If the input is reflected inside an HTML attribute (like a `value` attribute), strict attribute encoding must be used (e.g., converting `"` to `&quot;` and `'` to `&#x27;`) to prevent attackers from breaking out of the context.
2. **Input Sanitization and Validation:** Validate the `fname` and `lname` parameters on the server side using a strict allow-list approach, ensuring they only accept expected characters (like alphabetical characters for names). Do not rely on block-lists (like blocking the word "alert" or specific script tags), as these are easily bypassed using Unicode encoding and mixed-casing techniques.
3. **Implement a Content Security Policy (CSP):** Use a strict CSP HTTP response header to restrict the execution of unauthorized or inline scripts, which would neutralize this payload even if the injection flaw remains.
