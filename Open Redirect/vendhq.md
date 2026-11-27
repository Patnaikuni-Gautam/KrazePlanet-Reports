## Title
Path-Based Open Redirect - Vend

## Description
**Affected URL:** https://vendhq.pwnedbox.site/

A Path-Based Open Redirect vulnerability exists in the Vend web application. When multiple leading forward slashes followed by an external domain (`///bing.com`) are appended to the root URL path, the server returns an HTTP `302 Found` response with a protocol-relative `Location` header (`///bing.com/`). Modern browsers normalize the leading slashes and interpret the `Location` value as an external URL, redirecting victims off-site.

## Steps to Reproduce
1. Navigate to `https://vendhq.pwnedbox.site/` to confirm the standard landing page loads normally with a `200` status code.
2. Append `///bing.com` to the root URL to form `https://vendhq.pwnedbox.site///bing.com`.
3. Open the crafted URL in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: ///bing.com/`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://vendhq.pwnedbox.site///bing.com`

**HTTP Request:**
```http
GET ///bing.com HTTP/2
Host: vendhq.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: ///[bing.com/](https://bing.com/)
```

**Visual Evidence:**
- Baseline request to `https://vendhq.pwnedbox.site/` loading normally with a `200` status code.
  <img width="1921" height="962" alt="image" src="https://github.com/user-attachments/assets/c05218d9-3ae5-4fea-a721-dd80b11d8763" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: ///bing.com/`.
  <img width="1568" height="482" alt="image" src="https://github.com/user-attachments/assets/0584fc15-4ce1-4575-824b-76fae298ccb3" /><br><br>

- Browser navigating to `https://vendhq.pwnedbox.site///bing.com` and landing directly on `www.bing.com`.<br>
 <img width="567" height="50" alt="Screenshot 2026-11-27 161532" src="https://github.com/user-attachments/assets/20f16068-9835-4a84-8fca-e68c5773ad9f" /><br>
 <img width="1696" height="500" alt="Screenshot 2026-11-27 161539" src="https://github.com/user-attachments/assets/afce558a-b1c0-47dd-a341-dc871b5c81a5" />


## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `vendhq.pwnedbox.site` to lure users onto lookalike login pages and steal credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Normalize Request Paths:** Collapse consecutive leading slashes on the server side before constructing redirect targets.
2. **Block Protocol-Relative Redirects:** Ensure relative `Location` headers strictly begin with a single `/` followed by a valid path character, explicitly rejecting patterns starting with `//` or `/\`.
3. **Validate Destination Host:** If absolute URLs are used in redirects, parse the final URL and verify that the host strictly matches `vendhq.pwnedbox.site` before issuing the `302` response.
