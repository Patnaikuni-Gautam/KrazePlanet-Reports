## Title
Stored XSS in Support Ticket Subject & Description - DigitalOcean Support Center

## Description
**Affected URL:** https://digitalocean.pwnedbox.site/index.php?view=create

A Stored Cross-Site Scripting (XSS) vulnerability exists in the **Create Support Ticket** feature. Input submitted in both the **Subject** and **Description** fields is stored without sanitization and rendered as raw HTML on the **Support Tickets** page (`?view=tickets`), allowing persistent JavaScript execution whenever the ticket list is viewed.

## Steps to Reproduce
1. Navigate to `https://digitalocean.pwnedbox.site/index.php?view=create` to open the **Create Support Ticket** form.
2. Enter `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>` in the **Subject** field and `gt2'"<><ImG sRc=x oNeRroR=confirm(2)>` in the **Description** field.
3. Click **Submit Support Case** to save the ticket.
4. Open the **Support Tickets** page (`?view=tickets`) and observe both `confirm(1)` and `confirm(2)` executing immediately upon load.

## POC (Proof of Concept)
**Payloads:**
- **Subject:** `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>`
- **Description:** `gt2'"<><ImG sRc=x oNeRroR=confirm(2)>`

**Visual Evidence:**
- Payloads submitted in the **Subject** and **Description** fields of the **Create Support Ticket** form.
  <img width="1647" height="792" alt="image" src="https://github.com/user-attachments/assets/97f040d7-171f-4846-8c7a-d693d1ccb1c2" /><br><br>

- Persistent `confirm(1)` and `confirm(2)` dialogs executing sequentially on `digitalocean.pwnedbox.site/index.php?view=tickets`.
  <img width="1291" height="633" alt="image" src="https://github.com/user-attachments/assets/2044aec2-02d1-40a9-8292-d64c8665e509" />
  <img width="1337" height="605" alt="image" src="https://github.com/user-attachments/assets/f27bba70-d1ad-4c67-bc6f-28909f18e584" /><br><br>

- The **Open Tickets** list rendering both injected `<img>` tags as raw HTML (visible as broken image icons next to `gt1'"<>` and `gt2'"<>`).
  <img width="907" height="560" alt="image" src="https://github.com/user-attachments/assets/7c9f7d11-bc60-4558-a47b-b92347c7c9d9" /><br><br>


## Impact
Because the payloads are stored persistently in support tickets, any user or support administrator who views the tickets page automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over user or admin accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as reading, modifying, or closing support tickets.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect users or support operators to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities (e.g., using `htmlspecialchars()` in PHP) before rendering ticket subjects and descriptions.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and event handlers from support ticket submissions on the server side before storing them in the database.
3. **Avoid Unsafe DOM Rendering:** Ensure ticket fields are rendered strictly as plain text rather than raw HTML.
