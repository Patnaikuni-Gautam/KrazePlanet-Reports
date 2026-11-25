## Title
Stored XSS in Network Report Name - Ads Dashboard

## Description
**Affected URL:** https://ads.pwnedbox.site/

A Stored Cross-Site Scripting (XSS) vulnerability exists in the **New Network Report** feature. User input submitted in the **Report Name** field is stored without sanitization and rendered as raw HTML in the saved reports table, allowing persistent JavaScript execution whenever the reports list is viewed.

## Steps to Reproduce
1. Navigate to `https://ads.pwnedbox.site/` and open the **New Network Report** modal.
2. Enter `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>` into the **Report Name** field and select the desired options (e.g., **AdMob**, **Last 7 days**).
3. Click **Run & Save Report** to save the report.
4. Observe the `confirm(1)` dialog executing immediately when the saved reports table renders the newly created entry.

## POC (Proof of Concept)
**Payload:** `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>`

**Visual Evidence:**
- Payload submitted in the **Report Name** field of the **New Network Report** modal.
  <img width="492" height="472" alt="image" src="https://github.com/user-attachments/assets/5d4e7ff3-ca58-4c01-8dcb-e123a3f5de34" /><br><br>

- Persistent `confirm(1)` execution triggered on `ads.pwnedbox.site`, with the injected `<img>` tag rendered as raw HTML in the **Report Name** column (visible as a broken image icon next to `gt1'"<>`).
  <img width="1317" height="425" alt="image" src="https://github.com/user-attachments/assets/b74549a8-6770-4920-8f95-0d452018f3c5" />


## Impact
Because the payload is stored persistently in the shared network reports table, any team member or administrator who views the dashboard automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as modifying ad campaigns or deleting reports.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect users to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities before rendering report names in the dashboard table.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and event handlers from report submissions on the server side before storing them in the database.
3. **Avoid Unsafe DOM Rendering:** Render report names strictly as plain text (e.g., using `textContent`) rather than raw HTML.
