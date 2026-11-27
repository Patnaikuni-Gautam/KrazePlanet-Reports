## Title
Open Redirect in `/llink` `url` Parameter - Slack (Acme Login)

## Description
**Affected URL:** https://validation.pwnedbox.site/llink?url=/messages

An Open Redirect vulnerability exists in the Slack (Acme) sign-in flow's `/llink` endpoint. During login redirection, the `url` GET parameter accepts arbitrary external URLs without validating the destination domain, causing the server to return an HTTP `302 Found` response with a user-controlled `Location` header that redirects victims off-site.

## Steps to Reproduce
1. Navigate to `https://validation.pwnedbox.site/` and attempt to sign in while monitoring network traffic to observe the redirect chain (`/?redir=llink?url=/messages` routing to `/llink?url=/messages`).
2. Modify the `url` query parameter on the `/llink` endpoint to an arbitrary external domain such as `https://bing.com`.
3. Open `https://validation.pwnedbox.site/llink?url=https://bing.com` in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://validation.pwnedbox.site/llink?url=https://bing.com`

**HTTP Request:**
```http
GET /llink?url=[https://bing.com](https://bing.com) HTTP/2
Host: validation.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com](https://bing.com)
```

**Visual Evidence:**
- Network tab showing the login flow initiating a `302` redirect through `llink?url=/messages`.
  <img width="1741" height="1069" alt="image" src="https://github.com/user-attachments/assets/96a35162-b0f9-43d7-bd45-850d82c3f509" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: https://bing.com`.
  <img width="1557" height="482" alt="image" src="https://github.com/user-attachments/assets/4397e697-e6d6-47ca-b626-c029ac8b97df" /><br><br>

- Browser navigating to `https://validation.pwnedbox.site/llink?url=https://bing.com` and landing directly on `www.bing.com`.<br>
  <img width="987" height="53" alt="image" src="https://github.com/user-attachments/assets/3a29f775-261f-49af-82fa-01d6aad73395" /><br>
  <img width="1024" height="361" alt="image" src="https://github.com/user-attachments/assets/3db22e33-7952-4d56-be1f-5d944f236fa8" />



## Impact
Because the redirect originates from a trusted authentication domain, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `validation.pwnedbox.site` to lure users onto lookalike workspace sign-in pages and harvest credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in authentication links shared from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Restrict to Relative Paths:** Ensure the `url` parameter on `/llink` only accepts relative internal paths (e.g., `/messages`) that begin with a single `/` and do not start with protocol-relative prefixes (`//` or `/\`).
2. **Enforce a Strict Domain Allow-List:** If absolute URLs are required, parse the `url` value on the server side and verify that the scheme is `https` and the host strictly matches `validation.pwnedbox.site` before issuing the `302` response.
3. **Fallback to a Safe Default:** If the supplied `url` value fails validation, ignore it and redirect the user to a hardcoded safe default such as `/messages`.
