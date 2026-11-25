## Title
Basic URL Parameter Redirect - DevPulse

## Description
**Affected URL:** https://redirect.pwnedbox.site/

An Open Redirect vulnerability exists in the DevPulse application's outbound link handler. The `url` GET parameter accepts arbitrary external URLs without validating the destination domain, causing the server to return an HTTP `302 Found` response with a user-controlled `Location` header that redirects victims to any external site.

## Steps to Reproduce
1. Navigate to `https://redirect.pwnedbox.site/` and click an outbound story link to observe the redirect endpoint (`/?url=https%3A%2F%2Fwww.postgresql.org%2Fabout%2Fnews%2F`).
2. Modify the `url` query parameter to an arbitrary external domain such as `https://bing.com`.
3. Open `https://redirect.pwnedbox.site/?url=https://bing.com` in a browser or intercept the request in a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://redirect.pwnedbox.site/?url=https://bing.com`

**HTTP Request:**
```http
GET /?url=[https://bing.com](https://bing.com) HTTP/2
Host: redirect.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com](https://bing.com)
```

**Visual Evidence:**
- Network tab showing the legitimate outbound link routing through the `?url=` parameter with a `302` redirect status.
  <img width="1138" height="895" alt="image" src="https://github.com/user-attachments/assets/507b4552-b3dd-4e67-ac2d-6c33d9d50349" /><br><br>

- Intruder results confirming `302 Found` responses with `Location: https://bing.com` across unvalidated redirect payloads.
  <img width="1561" height="897" alt="image" src="https://github.com/user-attachments/assets/8ee61b5c-a076-4861-84de-f9bec39d8fbf" /><br><br>

- Browser navigating to `https://redirect.pwnedbox.site/?url=https://bing.com` and landing directly on `https://www.bing.com/`.
  <img width="943" height="77" alt="Screenshot 2026-11-25 230234" src="https://github.com/user-attachments/assets/dcfaa016-ca60-47ed-82c7-66cceabe4926" /><br>

  <img width="1847" height="951" alt="Screenshot 2026-11-25 230246" src="https://github.com/user-attachments/assets/53ebc0ff-a8a4-4e5f-825d-539221bc6961" />


## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `redirect.pwnedbox.site` to lure users onto lookalike login pages and steal credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive OAuth tokens/Referer data in broader attack chains.

## Remediation
1. **Enforce a Strict Domain Allow-List:** Validate the `url` parameter on the server side against an explicit allow-list of trusted domains and schemes (`https://` only), rejecting any unmatched or malformed URLs.
2. **Use Indirect Mapping:** Replace raw URL parameters with server-side lookup IDs (e.g., `?story_id=1`) that resolve to predefined destination URLs stored in the database.
3. **Add an Interstitial Warning Page:** If arbitrary external redirects are required by design, display a confirmation page clearly showing the destination URL before redirecting the user.
