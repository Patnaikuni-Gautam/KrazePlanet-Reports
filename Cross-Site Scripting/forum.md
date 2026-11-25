## Title
Stored XSS in Profile Signature - Acronis Forum

## Description
**Affected URL:** https://forum.pwnedbox.site/

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Acronis Forum **Edit Profile** page. User input entered into the **Signature** field is saved without proper sanitization and rendered directly as raw HTML, allowing persistent JavaScript execution whenever the signature is previewed or viewed.

## Steps to Reproduce
1. Navigate to `https://forum.pwnedbox.site/` and open **My Profile** from the left navigation menu.
2. Scroll down to the **Profile Signature** section and locate the **Signature** input box.
3. Paste `'"<><ImG sRc=x oNeRroR=confirm(1)>` into the Signature field and click **Save Profile**.
4. Observe the `confirm(1)` dialog executing immediately when the saved signature is rendered.

## POC (Proof of Concept)
**Payload:** `'"<><ImG sRc=x oNeRroR=confirm(1)>`

**Visual Evidence:**
* Payload injected into the **Signature** field, rendering as raw HTML in the Live Preview.
  <img width="1052" height="797" alt="image" src="https://github.com/user-attachments/assets/27b3c230-6caa-4121-8675-f942752705db" /><br><br>
* Persistent `confirm(1)` execution triggered on `forum.pwnedbox.site` after saving the profile.
  <img width="1177" height="791" alt="image" src="https://github.com/user-attachments/assets/08733937-f938-450d-99e9-3055295a7e92" />


## Impact
Because the payload is stored persistently in the profile signature, any user or administrator who views the profile or posts displaying the signature automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as modifying profiles or posting content.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect visitors to external phishing or malware sites.

## Remediation
1. **Sanitize HTML Input:** Filter all user-supplied HTML in the signature field through a strict allow-list sanitizer (e.g., DOMPurify) on both the server and client before saving or rendering.
2. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities whenever raw HTML formatting is not strictly required.
3. **Avoid Unsafe DOM Rendering:** Replace direct raw HTML or `innerHTML` rendering with safe DOM APIs (such as `textContent`) or sanitized template rendering to strip executable tags and event handlers like `onerror`.
