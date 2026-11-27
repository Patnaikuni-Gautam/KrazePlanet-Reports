## Title
Open Redirect in Logout `returnTo` Parameter - Zaption

## Description
**Affected URL:** https://zaption.pwnedbox.site/logout?returnTo=/

An Open Redirect vulnerability exists in the Zaption application's logout endpoint. The `returnTo` GET parameter accepts arbitrary external URLs without validating the destination domain, causing the server to return an HTTP `302 Found` response with a user-controlled `Location` header that redirects victims to any external site upon signing out.

## Steps to Reproduce
1. Navigate to `https://zaption.pwnedbox.site/`, open the user profile dropdown, and click **Sign out** while monitoring network traffic to observe the redirect endpoint (`/logout?returnTo=/`).
2. Modify the `returnTo` query parameter to an arbitrary external domain such as `https://bing.com`.
3. Open `https://zaption.pwnedbox.site/logout?returnTo=https://bing.com` in a browser or send the request through a proxy tool.
4. Observe the server responding with `HTTP/2 302 Found` and `Location: https://bing.com`, immediately redirecting the browser to `https://www.bing.com/`.

## POC (Proof of Concept)
**Payload URL:** `https://zaption.pwnedbox.site/logout?returnTo=https://bing.com`

**HTTP Request:**
```http
GET /logout?returnTo=[https://bing.com](https://bing.com) HTTP/2
Host: zaption.pwnedbox.site
```

**HTTP Response:**
```http
HTTP/2 302 Found
Location: [https://bing.com](https://bing.com)
```

**Visual Evidence:**
- Network tab showing the standard sign-out action routing through `logout?returnTo=/` with a `302` redirect status.
  <img width="1918" height="952" alt="image" src="https://github.com/user-attachments/assets/dc6bc9c2-7b67-4a17-bd24-73c0f823d315" /><br><br>

- Proxy capture confirming the `302 Found` response with `Location: https://bing.com`.
  <img width="1868" height="481" alt="image" src="https://github.com/user-attachments/assets/6fc4a8aa-8df4-4b0e-a840-2ce6affd17ff" /><br><br>

- Browser navigating to `https://zaption.pwnedbox.site/logout?returnTo=https://bing.com` and landing directly on `www.bing.com`.
  <img width="680" height="68" alt="Screenshot 2026-11-27 133540" src="https://github.com/user-attachments/assets/222618d5-a07c-4d5b-b6e4-b2992e472a68" /><br>
  <img width="1880" height="432" alt="Screenshot 2026-11-27 133547" src="https://github.com/user-attachments/assets/98e60c13-e826-4b80-b8ee-97162e6e7027" />



## Impact
Because the redirect originates from a trusted domain and occurs during a logout flow, attackers can abuse it in social engineering campaigns:
- **Phishing Attacks:** Craft convincing links using `zaption.pwnedbox.site` that log the user out and immediately redirect them to a lookalike login page to harvest credentials.
- **Malware Distribution:** Redirect unsuspecting visitors to external sites hosting malicious downloads or browser exploits.
- **Trust & Reputation Damage:** Undermine user confidence in authentication and logout links from the application.
- **Chained Exploitation:** Potentially leverage the open redirect to bypass URL filters or leak sensitive tokens/Referer data in broader attack chains.

## Remediation
1. **Restrict to Relative Paths:** Ensure the `returnTo` parameter only accepts relative internal paths (e.g., `/`, `/lessons`) that begin with a single `/` and do not start with protocol-relative prefixes (`//` or `/\`).
2. **Enforce a Strict Domain Allow-List:** If absolute URLs are required, parse the `returnTo` value on the server side and verify that the scheme is `https` and the host strictly matches `zaption.pwnedbox.site` before issuing the `302` response.
3. **Fallback to a Safe Default:** If the supplied `returnTo` value fails validation, ignore it and redirect the user to a hardcoded safe default such as `/`.
````
