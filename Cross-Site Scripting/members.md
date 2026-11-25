## Title
Stored XSS in Product Review Comments - Flipkart (Members)

## Description
**Affected URL:** https://members.pwnedbox.site/index.php?product_id=2

A Stored Cross-Site Scripting (XSS) vulnerability exists in the **Write a Customer Review** feature on the product page. While fields such as the reviewer's username are properly escaped, input submitted in the **Detailed Review Comments** field is stored without sanitization and rendered as raw HTML under **Customer Feedbacks**, allowing persistent JavaScript execution whenever the product page is viewed.

## Steps to Reproduce
1. Navigate to `https://members.pwnedbox.site/index.php?product_id=2` while logged in.
2. Under **Write a Customer Review**, select a star rating and enter a headline (e.g., `Testing stored xss`).
3. Paste `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>` into the **Detailed Review Comments** box and click **Submit Product Review**.
4. Observe the `confirm(1)` dialog executing immediately when the page reloads and renders the review under **Customer Feedbacks**.

## POC (Proof of Concept)
**Payload:** `gt1'"<><ImG sRc=x oNeRroR=confirm(1)>`


**Visual Evidence:**
- Payload submitted in the **Detailed Review Comments** field of the product review form.
  <img width="1452" height="885" alt="image" src="https://github.com/user-attachments/assets/f19939f2-fcd1-4215-b05a-3f0b4a4ea5de" /><br><br>

- Persistent `confirm(1)` execution triggered on `members.pwnedbox.site` upon page load.
  <img width="1010" height="665" alt="image" src="https://github.com/user-attachments/assets/dd1f4a29-dbc9-4a95-8465-515a6142fe22" /><br><br>

- The **Customer Feedbacks** section rendering the injected `<img>` tag as raw HTML (visible as a broken image icon next to `gt1'"<>`).
  <img width="627" height="180" alt="image" src="https://github.com/user-attachments/assets/18f436d9-4912-4ab1-b669-cc51ce17c138" />

## Impact
Because the payload is stored persistently on a public product page, any user or administrator who views the product automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as submitting reviews or modifying account details.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect shoppers to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities (e.g., using `htmlspecialchars()` in PHP) before rendering review comments in the **Customer Feedbacks** section.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and attributes from review submissions on the server side before storing them in the database.
3. **Ensure Consistent Output Encoding:** Apply the same output encoding used for the reviewer name field uniformly across all user-controlled review fields.
