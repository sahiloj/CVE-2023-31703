# CVE-2023-31703 — Reflected XSS in eScan Management Console

## Overview

**CVE ID:** CVE-2023-31703  
**Vulnerability Type:** Reflected Cross-Site Scripting (XSS)  
**Affected Product:** Microworld Technologies eScan Management Console  
**Affected Version:** 14.0.1400.2281  
**Severity:** Medium  
**Discovered By:** Sahil Ojha  
**Disclosure Date:** 16 May 2023  

---

## Description

A reflected Cross-Site Scripting (XSS) vulnerability exists in the **Edit User** functionality of the Microworld Technologies eScan Management Console (version 14.0.1400.2281). The application fails to properly sanitize or encode user-supplied input passed via the `from` parameter in the URL before reflecting it back in the HTTP response.

An unauthenticated or authenticated remote attacker who can trick a logged-in administrator into clicking a crafted link can inject and execute arbitrary JavaScript in the context of the victim's browser session. This can lead to session cookie theft, account takeover, credential harvesting, or further client-side attacks.

---

## Affected Component

| Field             | Details                                                      |
|-------------------|--------------------------------------------------------------|
| **Vendor**        | Microworld Technologies                                      |
| **Product**       | eScan Management Console                                     |
| **Version**       | 14.0.1400.2281                                               |
| **Component**     | `editUserName` endpoint — `from` URL parameter               |
| **Vendor Site**   | https://www.escanav.com                                      |
| **Software Link** | https://cl.escanav.com/ewconsole.dll                         |
| **Platform**      | Windows                                                      |

---

## Technical Details

### Root Cause

The `from` parameter accepted by the `/ewconsole/ewconsole.dll/editUserName` endpoint is reflected directly into the HTML response body without any HTML encoding or output sanitization. This allows an attacker to break out of the HTML attribute context and inject a `<script>` tag or other event-handler payload that is executed immediately by the browser.

### Affected Endpoint

```
GET /ewconsole/ewconsole.dll/editUserName?usrid=<ID>&from=<PAYLOAD>&P=
```

### Attack Vector

- **Network** — exploitable remotely via a crafted URL  
- **Authentication** — victim must be authenticated as a console user; the attacker does **not** need credentials  
- **User Interaction** — required (victim must click the malicious link)

---

## Proof of Concept

### Prerequisites

1. Access to a live or local installation of eScan Management Console 14.0.1400.2281.
2. A logged-in admin or regular user session in the target browser.

### Steps to Reproduce

1. Log in to the eScan Management Console with valid credentials.

2. Navigate to the following baseline URL to confirm the `from` parameter is reflected:
   ```
   https://cl.escanav.com/ewconsole/ewconsole.dll/editUserName?usrid=4&from=banner&P=
   ```

3. Replace the `from` value with the XSS payload below and open the crafted URL in the victim's authenticated browser:
   ```
   https://cl.escanav.com/ewconsole/ewconsole.dll/editUserName?usrid=4&from="><script>alert(document.cookie)</script>banner&P=
   ```

4. The browser will execute the injected script and display an alert box containing the session cookie, confirming the vulnerability.

### Screenshot

![XSS Proof of Concept](https://github.com/sahiloj/CVE-2023-31703/blob/main/XSS%201.png)

---

## Impact

| Impact Category       | Description                                                                                     |
|-----------------------|-------------------------------------------------------------------------------------------------|
| **Session Hijacking** | An attacker can steal the authenticated user's session cookie and take over their account.      |
| **Account Takeover**  | With admin-level session cookies, full administrative control of the console can be gained.     |
| **Phishing**          | Injected scripts can redirect the user to a phishing page hosted by the attacker.               |
| **Defacement**        | The attacker can manipulate the visible content of the page for the targeted user.              |
| **Malware Delivery**  | Malicious scripts can be used to silently download or execute further payloads in the browser.  |

---

## Remediation

Microworld Technologies should apply the following mitigations:

1. **Output Encoding** *(primary defense)* — HTML-encode all user-controlled values before inserting them into HTML responses. At minimum, encode `<`, `>`, `"`, `'`, and `&`. This alone prevents the injected markup from being interpreted by the browser.
2. **Input Validation** *(defense-in-depth)* — Use allowlists to restrict parameter values to their expected format (e.g., alphanumeric strings for the `from` parameter) rather than attempting to denylist specific characters, which is easily bypassed.
3. **Content Security Policy (CSP)** — Deploy a strict CSP header (e.g., `Content-Security-Policy: default-src 'self'`) to limit the impact of any remaining XSS vectors.
4. **HttpOnly Cookies** — Mark session cookies with the `HttpOnly` flag to prevent JavaScript from accessing them even if an XSS payload is executed.

---

## References

- [CVE-2023-31703 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2023-31703)
- [Microworld Technologies eScan](https://www.escanav.com)
- [OWASP — Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)

---

## Disclaimer

This information is provided for educational and responsible disclosure purposes only. The author is not responsible for any misuse of this information. Always obtain proper authorization before testing any system you do not own.


