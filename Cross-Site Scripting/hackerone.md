## Title
Stored XSS in Vulnerability Report Title, Description & Impact - HackerOne

## Description
**Affected URL:** https://hackerone.pwnedbox.site/index.php?view=submit

A Stored Cross-Site Scripting (XSS) vulnerability exists in the **Submit Vulnerability Report** feature. While the report title is safely escaped in the left sidebar list, input submitted in the **Title**, **Description**, and **Impact** fields is stored without sanitization and rendered as raw HTML in the main report view (`?view=dashboard&report_id=5`), allowing persistent JavaScript execution whenever the report is opened.

## Steps to Reproduce
1. Navigate to `https://hackerone.pwnedbox.site/index.php?view=submit` to open the **Submit Vulnerability Report** form.
2. Enter `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>` in **Title**, `gt2'"<><ImG sRc=x oNeRroR=confirm(2)>` in **Description**, and `gt3'"<><ImG sRc=x oNeRroR=confirm(3)>` in **Impact** (selecting a Target Program and Severity).
3. Click **Submit Report** to save the submission.
4. Open the submitted report on the dashboard (`?view=dashboard&report_id=5`) and observe `confirm(1)`, `confirm(2)`, and `confirm(3)` executing sequentially upon load.

## POC (Proof of Concept)
**Payloads:**
- **Title:** `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>`
- **Description:** `gt2'"<><ImG sRc=x oNeRroR=confirm(2)>`
- **Impact:** `gt3'"<><ImG sRc=x oNeRroR=confirm(3)>`

**Visual Evidence:**
- Payloads submitted in the **Title**, **Description**, and **Impact** fields of the **Submit Vulnerability Report** form.
  <img width="807" height="946" alt="image" src="https://github.com/user-attachments/assets/70b647f8-73e8-4494-a240-271338abd643" /><br><br>

- Persistent `confirm(1)`, `confirm(2)`, and `confirm(3)` dialogs executing sequentially on `hackerone.pwnedbox.site/index.php?view=dashboard&report_id=5`.
  <img width="1401" height="715" alt="image" src="https://github.com/user-attachments/assets/8dc92e57-1391-47e2-a2a5-14ade93c1350" />
  <img width="1342" height="740" alt="image" src="https://github.com/user-attachments/assets/e7baf18d-506f-4ac9-babf-5ae4497dc98e" />
  <img width="1177" height="677" alt="image" src="https://github.com/user-attachments/assets/749448ae-e744-4258-b899-aa49e087139b" /><br><br>

- The report detail view rendering all three injected `<img>` tags as raw HTML (visible as broken image icons next to `gt1'"<>`, `gt2'"<>`, and `gt3'"<>`).
  <img width="880" height="606" alt="image" src="https://github.com/user-attachments/assets/67d2aaf7-64f8-4bef-a63f-fe412fdbae82" />


## Impact
Because the payloads are stored persistently in the submitted report, any triager, program member, or administrator who views the report automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over triager or admin accounts.
- **Unauthorized Actions:** Perform privileged actions on behalf of the victim, such as accepting reports, disclosing vulnerabilities, or awarding bounties.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect users or triagers to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities (e.g., using `htmlspecialchars()` in PHP) before rendering report titles, descriptions, and impact fields in the main view.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and event handlers from report submissions on the server side before storing them in the database.
3. **Ensure Consistent Output Encoding:** Apply the same output encoding used in the sidebar report list uniformly across all sections of the report detail view.
