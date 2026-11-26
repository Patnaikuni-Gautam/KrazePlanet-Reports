## Title
Open Redirect in URL Parameter - Semrush

## Description
**Affected URL:** https://out.pwnedbox.site/

An Open Redirect vulnerability exists in the application's outbound link handler. The `url` GET parameter accepts arbitrary external URLs without validating the destination domain, causing the server to return an HTTP `302 Found` response with a user-controlled `Location` header that redirects victims to any external site.

## Steps to Reproduce
1. Navigate to `https://out.pwnedbox.site/` and click an outbound **Visit Page** link to observe the redirect endpoint (`/?url=https%3A%2F%2Fwww.semrush.com%2Ffeatures%2F`).
2. Modify the `url` query parameter to an arbitrary external domain such as `https://bing.com`.
3. Open `https://out.pwnedbox.site/?url=https://bing.com` in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://out.pwnedbox.site/?url=https://bing.com`

**HTTP Request:**
```http
GET /?url=[https://bing.com](https://bing.com) HTTP/2
Host: out.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com](https://bing.com)
```

**Visual Evidence:**
- Network tab showing the legitimate outbound link routing through the `?url=` parameter with a `302` redirect status.
  <img width="1927" height="1031" alt="image" src="https://github.com/user-attachments/assets/96c44280-e2d8-4d9d-befa-d64ca70170aa" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: https://bing.com`.
  <img width="1577" height="465" alt="image" src="https://github.com/user-attachments/assets/88444304-f6f1-4923-a3cc-852f8bc331cc" /><br><br>

- Browser navigating to `out.pwnedbox.site/?url=https://bing.com` and landing directly on `www.bing.com`.
  <img width="512" height="35" alt="image" src="https://github.com/user-attachments/assets/26672ec2-528a-4ace-8bed-0c49a6d00db0" /><br>
  <img width="512" height="168" alt="image" src="https://github.com/user-attachments/assets/6936a789-b30d-4c95-ba99-974975e83408" />


## Impact
Because the redirect originates from a trusted domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `out.pwnedbox.site` to lure users onto lookalike login pages and steal credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Enforce a Strict Domain Allow-List:** Validate the `url` parameter on the server side against an explicit allow-list of trusted domains and schemes (`https://` only), rejecting any unmatched or malformed URLs.
2. **Use Indirect Mapping:** Replace raw URL parameters with server-side lookup IDs (e.g., `?target_id=1`) that resolve to predefined destination URLs stored in the database.
3. **Add an Interstitial Warning Page:** If arbitrary external redirects are required by design, display a confirmation page clearly showing the destination URL before redirecting the user.
