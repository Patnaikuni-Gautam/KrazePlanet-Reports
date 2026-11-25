## Title
Stored XSS in Organization / Institution Field - Community Hub

## Description
**Affected URL:** https://community.pwnedbox.site/index.php

A Stored Cross-Site Scripting (XSS) vulnerability exists in the Community Hub registration workflow. While fields like First Name, Middle Name, and Last Name are properly escaped, input submitted in the **Organization / Institution** field during account registration is saved without sanitization and rendered as raw HTML on the **User Profile** page, allowing persistent JavaScript execution.

## Steps to Reproduce
1. Navigate to `https://community.pwnedbox.site/index.php` to open the **Register** page.
2. Fill in the required account details and enter `gt5'"<><ImG sRc=x oNeRroR=confirm(5)>` into the **Organization / Institution** field.
3. Complete the CAPTCHA and click **Register Me !** to create the account.
4. Log in to the newly created account, navigate to **My Profile**, and observe the `confirm(5)` dialog executing immediately upon page load.

## POC (Proof of Concept)
**Payload:** `gt5'"<><ImG sRc=x oNeRroR=confirm(5)>`

**Visual Evidence:**
- Payload submitted in the **Organization / Institution** field on the registration page.
  <img width="1556" height="1032" alt="image" src="https://github.com/user-attachments/assets/fe78caac-4f80-4a66-bf33-0a13d653645a" /><br><br>

- Persistent `confirm(5)` execution triggered on `community.pwnedbox.site` when viewing the **User Profile** page, where the injected `<img>` tag is parsed as raw HTML under **Organization / Institution**.
  <img width="1457" height="972" alt="image" src="https://github.com/user-attachments/assets/7e2d8055-d037-4120-80e1-c052eb4f602b" />

## Impact
Because the payload is stored persistently in the user's profile data, any user or administrator who views the profile automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as modifying account settings or posting content.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect visitors to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities (e.g., using `htmlspecialchars()` in PHP) before rendering the **Organization / Institution** field on the profile page.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and attributes from registration fields on the server side before storing them in the database.
3. **Ensure Consistent Output Encoding:** Apply the same output encoding used for the name fields uniformly across all user-controlled profile fields.
