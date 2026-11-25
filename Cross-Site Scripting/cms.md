## Title
Stored XSS in Article Description HTML Editor - Quill CMS

## Description
**Affected URL:** https://cms.pwnedbox.site/

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Quill CMS article editor. Input entered via the "Edit HTML source" (`</>`) mode in the Description field is saved without sanitization and rendered directly via `innerHTML` on the "My Articles" page, allowing persistent JavaScript execution whenever the article is viewed.

## Steps to Reproduce
1. Navigate to `https://cms.pwnedbox.site/` and open the **Write** tab.
2. Enter a title (e.g., `GT`) and click the **Edit HTML source** (`</>`) button in the Description toolbar.
3. Paste `'"<><ImG sRc=x oNeRroR=confirm(1)>` into the HTML source box and publish the article.
4. Open the **My Articles** page and observe `confirm(1)` executing immediately upon load.

## POC (Proof of Concept)
**Payload:** `'"<><ImG sRc=x oNeRroR=confirm(1)>`[cite: 56]

**Visual Evidence:**
* Payload injected into the Description field via the "Edit HTML source" (`</>`) view.
  <img width="1332" height="561" alt="image" src="https://github.com/user-attachments/assets/9cb613de-e96a-4034-8e75-d4389e10403c" /><br><br>

  
* Persistent `confirm(1)` execution on the "My Articles" page after publishing.
  <img width="1112" height="657" alt="image" src="https://github.com/user-attachments/assets/1de048a3-3d2e-4c1b-9985-ab866fb963fb" />


## Impact
Because the payload is stored persistently, any user or administrator who views the article automatically executes the attacker's JavaScript:
* **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
* **Unauthorized Actions:** Perform actions on behalf of the victim, such as modifying or deleting articles.
* **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
* **Malicious Redirection:** Redirect visitors to external phishing or malware sites.

## Remediation
1. **Sanitize HTML Input:** Filter all rich-text and raw HTML through a strict allow-list sanitizer (e.g., DOMPurify) on both the server and client before saving or rendering.
2. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities whenever raw HTML formatting is not strictly required.
3. **Avoid Unsafe `innerHTML`:** Replace direct `innerHTML` rendering with safe DOM APIs (such as `textContent`) or sanitized template rendering to strip executable tags and event handlers like `onerror`.
