## Title
Open Redirect via Backslash & `@` Bypass in Logout Parameter - Tumblr (SSO)

## Description
**Affected URL:** https://sso.pwnedbox.site/

An Open Redirect vulnerability exists in the logout flow's `redirect_to` GET parameter. While the application attempts to restrict redirects to `www.tumblr.com`, its URL validation can be bypassed using a backslash and `@` sequence (`https://bing.com\@www.tumblr.com/dashboard`). The server parses the string as targeting `www.tumblr.com` and issues an HTTP `302 Found` response with the crafted `Location` header, which modern browsers normalize by converting `\` to `/` and redirecting the user to `bing.com`.

## Steps to Reproduce
1. Navigate to `https://sso.pwnedbox.site/` and click **Log out** while monitoring network traffic to observe the redirect endpoint (`/?action=logout&redirect_to=https://www.tumblr.com/dashboard`).
2. Modify the `redirect_to` parameter to `https://bing.com\@www.tumblr.com/dashboard`.
3. Open `https://sso.pwnedbox.site/?action=logout&redirect_to=https://bing.com\@www.tumblr.com/dashboard` in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com\@www.tumblr.com/dashboard`, immediately redirecting the browser to `https://www.bing.com/@www.tumblr.com/dashboard`.

## POC (Proof of Concept)
**Payload URL:** `https://sso.pwnedbox.site/?action=logout&redirect_to=https://bing.com\@www.tumblr.com/dashboard`

**HTTP Request:**
```http
GET /?action=logout&redirect_to=[https://bing.com](https://bing.com)\@[www.tumblr.com/dashboard](https://www.tumblr.com/dashboard) HTTP/2
Host: sso.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com](https://bing.com)\@[www.tumblr.com/dashboard](https://www.tumblr.com/dashboard)
```

**Visual Evidence:**
- Network tab showing the standard logout action routing through `/?action=logout&redirect_to=https://www.tumblr.com/dashboard` with a `302` redirect status.
  <img width="1924" height="1050" alt="image" src="https://github.com/user-attachments/assets/bac0528e-900f-41dc-92a1-e1de01d4efe6" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: https://bing.com\@www.tumblr.com/dashboard`.
  <img width="1877" height="525" alt="image" src="https://github.com/user-attachments/assets/cba03df4-5daa-41dd-a4ab-e75e4fbe5e83" /><br><br>

- Browser navigating to `https://sso.pwnedbox.site/?action=logout&redirect_to=https://bing.com\@www.tumblr.com/dashboard` and landing directly on `www.bing.com/@www.tumblr.com/dashboard`.
  <img width="512" height="34" alt="image" src="https://github.com/user-attachments/assets/e2e601dd-bb9d-40d3-9fe9-7369d5ec35a4" /><br>
  <img width="512" height="158" alt="image" src="https://github.com/user-attachments/assets/012864e5-8493-49aa-8aa6-eeb703af4df0" />


## Impact
Because the redirect originates from a trusted SSO domain and occurs during a logout flow, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `sso.pwnedbox.site` that log the user out and immediately redirect them to a lookalike login page to harvest credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in authentication and logout links from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Use a Strict, Standards-Compliant URL Parser:** Parse the `redirect_to` value using a WHATWG-compliant URL parser on the server side and explicitly verify that the extracted `hostname` strictly equals `www.tumblr.com` (and that the scheme is `https`).
2. **Reject Backslashes and Credentials in URLs:** Explicitly reject any `redirect_to` value containing backslashes (`\`), `@` symbols, or userinfo components before issuing a redirect.
3. **Prefer Relative Paths:** Restrict `redirect_to` to relative internal paths (e.g., `/dashboard`) that begin with a single `/` and do not contain protocol-relative prefixes (`//`).
