## Title
Stored XSS in Interactive Poll Question and Description - Community Hub

## Description
**Affected URL:** https://articles.pwnedbox.site/index.php

A Stored Cross-Site Scripting (XSS) vulnerability exists in the **Create Interactive Poll** feature under the **Events & Polls** portal. While the poll options and creator username are properly escaped, input submitted in the **Poll Question** and **Poll Description / Context** fields is saved without sanitization and rendered as raw HTML under **Active Polls**, allowing persistent JavaScript execution whenever the page is viewed.

## Steps to Reproduce
1. Navigate to `https://articles.pwnedbox.site/index.php`, open **Events & Polls**, and select the **Interactive Polls** tab.
2. In the **Create Interactive Poll** form, enter `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>` into **Poll Question** and `gt2'"<><ImG sRc=x oNeRroR=confirm(2)>` into **Poll Description / Context**.
3. Fill in at least two poll options and click **Publish Poll**.
4. Observe the `confirm(1)` and `confirm(2)` dialogs executing immediately upon reload when the poll is rendered under **Active Polls**.

## POC (Proof of Concept)
**Payloads:**
- **Poll Question:** `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>`
- **Poll Description / Context:** `gt2'"<><ImG sRc=x oNeRroR=confirm(2)>`

**Visual Evidence:**
- Payloads submitted in the **Poll Question** and **Poll Description / Context** fields of the **Create Interactive Poll** form.
  <img width="1345" height="857" alt="image" src="https://github.com/user-attachments/assets/2acc1a18-891a-4e70-8af9-6053609fdf14" /><br><br>

- Persistent `confirm(1)` and `confirm(2)` dialogs executing on `articles.pwnedbox.site` immediately after publishing the poll.
  <img width="1317" height="856" alt="image" src="https://github.com/user-attachments/assets/103c6320-e55a-45c9-a76d-9ea25a0a030d" />
  <img width="1295" height="741" alt="image" src="https://github.com/user-attachments/assets/37d1e433-e415-4467-9294-8649d1d742e2" /><br><br>

- The **Active Polls** list rendering both injected `<img>` tags as raw HTML (visible as broken image icons next to `gt1'"<>` and `gt2'"<>`), while the poll options remain safely escaped.
  <img width="750" height="336" alt="image" src="https://github.com/user-attachments/assets/914c213a-c867-4066-a1fe-12ed90171ad5" />


## Impact
Because the payloads are stored persistently on the public polls page, any user or administrator who visits the **Events & Polls** portal automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as creating polls, posting discussions, or modifying account settings.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect visitors to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities (e.g., using `htmlspecialchars()` in PHP) before rendering the **Poll Question** and **Poll Description / Context** fields.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and attributes from poll submissions on the server side before storing them in the database.
3. **Ensure Consistent Output Encoding:** Apply the same output encoding already used for the poll option fields uniformly across all user-controlled poll fields.
