## Title
Path-Based Open Redirect - HackerOne (Hacktivity)

## Description
**Affected URL:** https://stranger.pwnedbox.site/

A Path-Based Open Redirect vulnerability exists in the HackerOne Hacktivity application. When a crafted slash and backslash sequence (`//\/bing.com/`) is appended to the root URL path, the server returns an HTTP `302 Found` response with a protocol-relative `Location` header (`//\/bing.com/`). Modern browsers normalize the slashes and interpret the `Location` value as an external URL, redirecting victims off-site.

## Steps to Reproduce
1. Navigate to `https://stranger.pwnedbox.site/` to confirm the standard page loads normally with a `200` status code.
2. Append the path-based redirect payload `/\/bing.com/` to the domain to form `https://stranger.pwnedbox.site//\/bing.com/`.
3. Open the crafted URL in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: //\/bing.com/`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://stranger.pwnedbox.site//\/bing.com/`

**HTTP Request:**
```http
GET //\/[bing.com/](https://bing.com/) HTTP/2
Host: stranger.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: //\/[bing.com/](https://bing.com/)
```

**Visual Evidence:**
- Baseline request to `https://stranger.pwnedbox.site/` loading normally with a `200` status code.
  <img width="1907" height="943" alt="image" src="https://github.com/user-attachments/assets/5834f491-38eb-4ce8-82cd-4fafc581c60a" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: //\/bing.com/`.
  <img width="1857" height="386" alt="image" src="https://github.com/user-attachments/assets/12198d40-3fe7-4223-9aad-7bba1332f8b8" /><br><br>

- Browser navigating to `https://stranger.pwnedbox.site//\/bing.com/` and landing directly on `www.bing.com`.
  <img width="586" height="67" alt="Screenshot 2026-11-27 131411" src="https://github.com/user-attachments/assets/82f868f1-6332-4fb5-ac9d-8e012417010b" /><br>
  <img width="1733" height="603" alt="Screenshot 2026-11-27 131432" src="https://github.com/user-attachments/assets/00d2dd77-3e4b-4eb6-a624-62da9ced08d3" />



## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `stranger.pwnedbox.site` to lure users onto lookalike login pages and steal credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Normalize Request Paths:** Collapse consecutive slashes and reject or strip backslashes (`\` and `%5C`) on the server side before constructing redirect targets.
2. **Block Protocol-Relative Redirects:** Ensure relative `Location` headers strictly begin with a single `/` followed by a valid path character, explicitly rejecting patterns starting with `//` or `/\`.
3. **Validate Destination Host:** If absolute URLs are used in redirects, parse the final URL and verify that the host strictly matches `stranger.pwnedbox.site` before issuing the `302` response.
