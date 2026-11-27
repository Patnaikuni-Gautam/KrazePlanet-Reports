## Title
Open Redirect in `/redirect` `url` Parameter - HackerOne (Live Hacking Events)

## Description
**Affected URL:** https://hackevents.pwnedbox.site/redirect?url=https%3A%2F%2Fregister.hackerone-events.com

An Open Redirect vulnerability exists in the HackerOne Live Hacking Events application's `/redirect` endpoint. The `url` GET parameter accepts arbitrary external URLs without validating the destination domain against an allow-list, causing the server to return an HTTP `302 Found` response with a user-controlled `Location` header that redirects victims off-site.

## Steps to Reproduce
1. Navigate to `https://hackevents.pwnedbox.site/` and click **Register now** (or **Register**) while monitoring network traffic to observe the redirect endpoint (`/redirect?url=https%3A%2F%2Fregister.hackerone-events.com`).
2. Modify the `url` query parameter on the `/redirect` endpoint to an arbitrary external domain such as `https://bing.com`.
3. Open `https://hackevents.pwnedbox.site/redirect?url=https://bing.com` in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://hackevents.pwnedbox.site/redirect?url=https://bing.com`

**HTTP Request:**
```http
GET /redirect?url=[https://bing.com](https://bing.com) HTTP/2
Host: hackevents.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com](https://bing.com)
```

**Visual Evidence:**
- Network tab showing the registration button routing through `redirect?url=https%3A%2F%2Fregister.hackerone-events.com` with a `302` redirect status.
  <img width="1904" height="987" alt="image" src="https://github.com/user-attachments/assets/c8a8aa38-087a-40be-921e-badd3089c38c" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: https://bing.com`.
  <img width="1570" height="477" alt="image" src="https://github.com/user-attachments/assets/c1d9b3f1-454c-4e59-b406-d1b39d98ff4f" /><br><br>

- Browser navigating to `https://hackevents.pwnedbox.site/redirect?url=https://bing.com` and landing directly on `www.bing.com`.<br>
  <img width="702" height="43" alt="Screenshot 2026-11-27 151241" src="https://github.com/user-attachments/assets/af66e7f8-f409-4f46-abdf-9753668a28cb" /><br>
  <img width="1024" height="361" alt="image" src="https://github.com/user-attachments/assets/c07d7f17-d9f3-468f-92e1-940dbb16f13b" />



## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `hackevents.pwnedbox.site` to lure users onto lookalike event registration or login pages to harvest credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in event registration links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Enforce a Strict Domain Allow-List:** Parse the `url` parameter on the server side using a standards-compliant URL parser and verify that the scheme is `https` and the host strictly matches approved domains (such as `register.hackerone-events.com` or `hackevents.pwnedbox.site`) before issuing the `302` response.
2. **Use Indirect Mapping:** Replace raw destination URLs in the `url` parameter with fixed server-side identifiers (e.g., `/redirect?target=register`) mapped to hardcoded destinations.
3. **Fallback to a Safe Default:** If the supplied `url` value fails validation, ignore it and redirect the user to a hardcoded safe default such as `/`.
