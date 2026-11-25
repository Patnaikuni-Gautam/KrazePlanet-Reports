## Title
Stored XSS in Movie Reactions & Comments - Netflix

## Description
**Affected URL:** https://netflix.pwnedbox.site/103

A Stored Cross-Site Scripting (XSS) vulnerability exists in the **How are you feeling about this movie?** reaction form on movie detail pages. User input submitted in the **Share Your Thoughts & Reactions** field is stored without sanitization and rendered as raw HTML under **Viewer Reactions**, allowing persistent JavaScript execution whenever the movie page is viewed.

## Steps to Reproduce
1. Navigate to `https://netflix.pwnedbox.site/103` (e.g., *The Dark Knight* movie page).
2. Under **How are you feeling about this movie?**, select a mood and star rating.
3. Paste `'"<><ImG sRc=x oNeRroR=confirm(1)>` into the **Share Your Thoughts & Reactions** box and click **Post Reaction & Feeling**.
4. Observe the `confirm(1)` dialog executing immediately when the page reloads and renders the comment under **Viewer Reactions**.

## POC (Proof of Concept)
**Payload:** `'"<><ImG sRc=x oNeRroR=confirm(1)>`

**Visual Evidence:**
- Payload submitted in the **Share Your Thoughts & Reactions** field on the movie detail page.
  <img width="810" height="812" alt="image" src="https://github.com/user-attachments/assets/75f1e9ae-f9b1-4ccf-b5b7-e2b5442102c5" /><br><br>

- Persistent `confirm(1)` execution triggered on `netflix.pwnedbox.site/103`, with the injected `<img>` tag parsed as raw HTML under **Viewer Reactions**.
  <img width="1332" height="633" alt="image" src="https://github.com/user-attachments/assets/f08bcad1-b072-4f20-a4a8-eca78e93bb8d" />


## Impact
Because the payload is stored persistently on a public movie page, any user or administrator who views the page automatically executes the attacker's JavaScript:
- **Session Hijacking:** Steal session cookies or authentication tokens to take over accounts.
- **Unauthorized Actions:** Perform actions on behalf of the victim, such as posting reactions or modifying account settings.
- **Phishing:** Manipulate the DOM to inject fake login prompts and harvest credentials.
- **Malicious Redirection:** Redirect viewers to external phishing or malware sites.

## Remediation
1. **Apply HTML Escaping:** Encode special HTML characters (`<`, `>`, `&`, `"`, `'`) into their corresponding HTML entities before rendering user comments in the **Viewer Reactions** section.
2. **Sanitize User Input:** Validate and strip unexpected HTML tags and event handlers from reaction submissions on the server side before storing them in the database.
3. **Avoid Unsafe DOM Rendering:** Render reaction comments strictly as plain text rather than raw HTML.
