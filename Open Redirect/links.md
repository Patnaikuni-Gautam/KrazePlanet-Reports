## Title
Path-Based Open Redirect - Omise Developer Blog

## Description
**Affected URL:** https://links.pwnedbox.site/index.php?category=developers&page=1

A Path-Based Open Redirect vulnerability exists in the Omise Developer Blog application. When crafted slash and backslash sequences (such as `//\/bing.com/`) are prepended to the URL path, the server returns an HTTP `302 Found` response with a protocol-relative `Location` header containing the attacker-controlled domain. Browsers normalize the slashes and treat the `Location` value as an external URL, redirecting victims off-site.

## Steps to Reproduce
1. Navigate to `https://links.pwnedbox.site/index.php?category=developers&page=1` to confirm the standard endpoint returns `200 OK`.
2. Prepend a path-based redirect payload (`/\/bing.com`) to the request path to form `https://links.pwnedbox.site//\/bing.com/index.php?category=developers&page=1`.
3. Open the crafted URL in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and a `Location` header pointing to `bing.com`, immediately redirecting the browser to `https://www.bing.com/index.php?category=developers&page=1`.

## POC (Proof of Concept)
**Payload URL:** `https://links.pwnedbox.site//\/bing.com/index.php?category=developers&page=1`

**HTTP Request:**
```http
GET //\/[bing.com/index.php?category=developers&page=1](https://bing.com/index.php?category=developers&page=1) HTTP/2
Host: links.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: //%5C%5C/[bing.com/index.php?category=developers&page=1](https://bing.com/index.php?category=developers&page=1)
```

**Visual Evidence:**
- Baseline request to `https://links.pwnedbox.site/index.php?category=developers&page=1` loading normally with a `200` status code.
  <img width="1916" height="871" alt="image" src="https://github.com/user-attachments/assets/10e39f34-0910-4f12-b163-1ffaee56aeb2" /><br><br>

- Proxy results confirming `302 Found` responses with `Location: //%5C%5C/bing.com/index.php?category=developers&page=1`.
  <img width="1557" height="972" alt="image" src="https://github.com/user-attachments/assets/edcc7b6a-0dd6-4b61-abe7-416d3e006f56" /><br><br>

- Browser navigating to `https://links.pwnedbox.site//\/bing.com/index.php?category=developers&page=1` and redirecting directly to `www.bing.com/index.php?category=developers&page=1`.
  <img width="887" height="86" alt="image" src="https://github.com/user-attachments/assets/9c6e76d7-f477-45e2-a7b2-50bbc859945c" /><br>
  <img width="1892" height="662" alt="image" src="https://github.com/user-attachments/assets/eb217afb-660e-4e1a-9a17-1e0358d606ff" />



## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `links.pwnedbox.site` to lure users onto lookalike login pages and steal credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Normalize Request Paths:** Collapse consecutive slashes and reject or strip backslashes (`\` and `%5C`) on the server side before constructing redirect targets.
2. **Block Protocol-Relative Redirects:** Ensure relative `Location` headers strictly begin with a single `/` followed by a valid path character, explicitly rejecting patterns starting with `//` or `/\`.
3. **Validate Destination Host:** If absolute URLs are used in redirects, parse the final URL and verify that the host strictly matches `links.pwnedbox.site` before issuing the `302` response.
