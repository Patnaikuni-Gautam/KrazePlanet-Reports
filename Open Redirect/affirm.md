## Title
Path-Based Open Redirect - Affirm

## Description
**Affected URL:** https://affirm.pwnedbox.site/?category=all&page=3

A Path-Based Open Redirect vulnerability exists in the Affirm web application. When multiple leading forward slashes followed by an external domain (`////bing.com`) are prepended to the URL path, the server returns an HTTP `302 Found` response with a protocol-relative `Location` header (`////bing.com?category=all&page=3`). Modern browsers normalize the leading slashes and treat the `Location` value as an external URL, redirecting victims off-site.

## Steps to Reproduce
1. Navigate to `https://affirm.pwnedbox.site/?category=all&page=3` to confirm the standard endpoint loads normally with a `200` status code.
2. Prepend `////bing.com` to the query path to form `https://affirm.pwnedbox.site////bing.com?category=all&page=3`.
3. Open the crafted URL in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: ////bing.com?category=all&page=3`, immediately redirecting the browser to `https://www.bing.com/?category=all&page=3`.

## POC (Proof of Concept)
**Payload URL:** `https://affirm.pwnedbox.site////bing.com?category=all&page=3`

**HTTP Request:**
```http
GET ////bing.com?category=all&page=3 HTTP/2
Host: affirm.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: ////bing.com?category=all&page=3
```

**Visual Evidence:**
- Baseline request to `https://affirm.pwnedbox.site/?category=all&page=3` loading normally with a `200` status code.
  <img width="1914" height="832" alt="image" src="https://github.com/user-attachments/assets/f3804eb9-ef15-458a-b7e1-bd70c81b649a" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: ////bing.com?category=all&page=3`.
  <img width="1560" height="485" alt="image" src="https://github.com/user-attachments/assets/009fec54-6d7a-4fed-aefe-9453d18697cf" /><br><br>

- Browser navigating to `https://affirm.pwnedbox.site////bing.com?category=all&page=3` and landing directly on `www.bing.com/?category=all&page=3`.<br><br>
  <img width="615" height="51" alt="Screenshot 2026-11-27 135411" src="https://github.com/user-attachments/assets/6e2cf823-2d69-4284-8dd7-ee7c761966ce" /><br>
  <img width="1570" height="465" alt="Screenshot 2026-11-27 135419" src="https://github.com/user-attachments/assets/b7611ba0-45bc-4480-b628-7544a582345c" />



## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `affirm.pwnedbox.site` to lure users onto lookalike login pages and steal credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Normalize Request Paths:** Collapse consecutive leading slashes on the server side before constructing redirect targets.
2. **Block Protocol-Relative Redirects:** Ensure relative `Location` headers strictly begin with a single `/` followed by a valid path character, explicitly rejecting patterns starting with `//` or `/\`.
3. **Validate Destination Host:** If absolute URLs are used in redirects, parse the final URL and verify that the host strictly matches `affirm.pwnedbox.site` before issuing the `302` response.
````[cite: 43, 44, 45, 46]
