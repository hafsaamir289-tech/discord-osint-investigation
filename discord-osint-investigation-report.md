# Discord Infrastructure Investigation — A Passive OSINT Report

**Author:** Hafsa Amir
**Date:** 23 July 2026
**Classification:** Public / Educational
**Methodology:** 100% Passive OSINT — no scanning, no exploitation, no interaction with non-public systems

---

## ⚠️ Ethical Scope & Disclaimer

This investigation was conducted entirely using **publicly available information** — DNS records, Certificate Transparency logs, public ASN/WHOIS data, and information Discord itself has published (engineering blog, job postings, public docs). At no point were any active scanning tools, intrusion techniques, authentication bypass attempts, or unauthorized access methods used against Discord's systems. This report is a learning exercise in reconnaissance methodology, submitted in the spirit of responsible, transparent security research.

---

## Executive Summary

This report documents a passive Open Source Intelligence (OSINT) investigation into Discord's public-facing internet infrastructure. The goal was to practice real-world reconnaissance methodology — the same first step a professional penetration tester or red teamer takes before any engagement — using only publicly accessible data sources. Findings span DNS configuration, email authentication (SPF/DMARC), TLS/certificate posture, network/cloud infrastructure, and technology stack fingerprinting.

## Scope

- DNS Analysis
- Email Security (SPF & DMARC)
- SSL/TLS & Certificate Transparency
- Network Infrastructure (ASN, cloud, CDN)
- Technology Fingerprinting

## Methodology

All data was collected using passive, publicly accessible tools and databases — no packets were sent to non-public endpoints and no authentication boundary was tested.

**Tools/Sources used:** Connected.app, NSLookup.io, DNSChecker, DNSLookup, DMARC Manager, Hurricane Electric BGP Toolkit, Cloudflare Radar, AppSec Santa, and Discord's own Engineering Blog.

---

## 1. DNS Analysis

**Primary domain:** `discord.com`

### A Records
| IP Address |
|---|
| 162.159.128.233 |
| 162.159.135.232 |
| 162.159.136.232 |
| 162.159.137.232 |
| 162.159.138.232 |

**Observation:** Multiple concurrent A records point to load-balanced, geographically redundant edge infrastructure — a pattern consistent with a service operating behind a CDN/anycast network rather than a single origin server.

### AAAA Records
No public IPv6 records identified for the primary domain.

### MX Records
| Priority | Mail Server |
|---:|---|
| 1 | aspmx.l.google.com |
| 5 | alt1.aspmx.l.google.com |
| 5 | alt2.aspmx.l.google.com |
| 10 | aspmx2.googlemail.com |
| 10 | aspmx3.googlemail.com |

**Observation:** Discord routes corporate email through Google Workspace rather than self-hosting mail infrastructure.

### TXT Records
Verification records were identified for Google, Apple, Slack, DocuSign, and Atlassian, alongside the SPF policy below — indicating the range of enterprise SaaS tools integrated into Discord's corporate domain.

### SPF Record
```
v=spf1 include:_spf.google.com include:mail.zendesk.com include:sendgrid.net include:3885857.spf06.hubspotemail.net ip4:198.2.180.60 -all
```
The `-all` qualifier enforces a **hard fail** — mail from unauthorized senders is explicitly rejected, not just flagged. Authorized senders include Google Workspace, Zendesk (support), SendGrid (transactional email), and HubSpot (marketing).

### DMARC Record
```
v=DMARC1; p=reject;
```
A `p=reject` policy is the strongest DMARC enforcement level available, offering strong protection against domain spoofing and business email compromise (BEC) attempts.

---

## 2. SSL/TLS & Certificate Transparency

- **Issuer:** Google Trust Services
- **Key Type:** ECDSA
- **Protocol:** TLS 1.3
- **SANs:** `discord.com`, `*.discord.com`

Certificate Transparency logs surfaced a range of legitimate subdomains, including:
- `app.discord.com`
- `blog.discord.com`
- `bugs.discord.com`
- `canary.discord.com`
- `click.discord.com`
- `androiddiag.discord.com`

**Observation:** CT logs are a double-edged sword — they're essential for public trust and auditability, but they also passively reveal a service's subdomain footprint to anyone watching (including attackers scouting for lower-profile, less-hardened endpoints like staging/canary/diagnostic subdomains).

---

## 3. Network Infrastructure

- **ASN:** AS49544
- **Organization:** i3D.net B.V.
- **Cloud providers (publicly reported):** AWS and Google Cloud Platform
- **CDN/Edge:** Cloudflare

## 4. Technology Fingerprinting

| Layer | Technologies |
|---|---|
| Frontend | JavaScript, React, Redux, Sass |
| Backend (publicly reported) | Elixir, Phoenix, PostgreSQL |
| Real-time | WebSockets, WebRTC, Opus codec |
| Architecture | Microservices, distributed systems |

---

## 5. Security Observations

1. Strong SPF (`-all` hard fail) and DMARC (`p=reject`) implementation — among the strictest enforcement levels available.
2. Modern TLS 1.3 across the primary domain and subdomains.
3. Cloudflare sits in front of origin infrastructure, providing CDN performance benefits and DDoS mitigation.
4. Enterprise email is fully outsourced to Google Workspace rather than self-hosted, reducing mail-server attack surface.
5. DNS infrastructure shows built-in redundancy via multiple A records rather than a single point of failure.

## 6. Interesting Findings

1. **DMARC is set to full enforcement (`reject`)** — many organizations leave DMARC at `p=none` (monitor-only) for years; Discord has moved to the strictest tier.
2. **Certificate Transparency logs reveal internal-sounding subdomains** (`canary`, `bugs`, `androiddiag`) that are technically public knowledge the moment a cert is issued — a reminder that CT logs are a standard part of any attacker's (or researcher's) initial recon, regardless of how "hidden" a subdomain feels.
3. **The email supply chain is more exposed than the product itself** — SPF reveals four separate third-party vendors (Google, Zendesk, SendGrid, HubSpot) authorized to send mail as Discord, meaning Discord's phishing-resistance partly depends on the security posture of those vendors too.

---

## Analyst Reflection

**Q: If you were a security analyst, what part of this infrastructure would you monitor most closely, and why?**

Certificate Transparency logs and DNS/subdomain enumeration would be my top priority to monitor continuously. New certificates are issued (and logged publicly) the moment a new subdomain goes live — including internal tools, staging environments, or diagnostic endpoints that were never meant to be public-facing. An attacker doesn't need to scan Discord's network to find these; they just need to watch CT logs in real time. A mature security team typically runs automated CT log monitoring so that any new `*.discord.com` certificate triggers an internal review before it becomes someone else's discovery. The second-closest area would be the SPF-authorized third-party vendor list — since a compromise at any one of those providers (Zendesk, SendGrid, HubSpot) could be leveraged to send spoofed mail that still passes SPF for discord.com.

**Q: What was the most challenging part of this investigation, and what did you learn?**

Honestly, every stage of this had its own learning curve as a first attempt at real reconnaissance. Reading raw DNS output and knowing what actually mattered (versus noise) took some getting used to — an A record dump or a TXT record list means nothing until you understand what each one is telling you about how a company runs its infrastructure. Making sense of SPF and DMARC syntax was its own hurdle; the qualifiers (`-all` vs `~all`, `p=reject` vs `p=none`) look like small details but represent very different security postures, and it took cross-referencing multiple sources to be confident I was reading them correctly. The biggest lesson, though, was learning where "passive OSINT" ends — understanding that pulling public DNS records, CT logs, and ASN data is fair game, but that the line into active reconnaissance (port scanning, probing endpoints, etc.) is a hard boundary I needed to respect even out of curiosity. Overall, this project taught me that reconnaissance is less about finding "hidden secrets" and more about learning to read what's already publicly visible — which is exactly why organizations like Discord invest in things like strict DMARC policies and CT log monitoring in the first place.

---

## Conclusion

Passive OSINT reconnaissance of Discord's public infrastructure shows a mature, security-conscious posture: strict email authentication, modern TLS, CDN-backed edge protection, and cloud-distributed architecture. No sensitive or non-public information was accessed at any point in this investigation — every data point above is derived from information Discord (or public internet registries) already makes visible to anyone who looks.

## References

- Connected.app
- NSLookup.io
- DNSChecker
- DNSLookup
- DMARC Manager
- Hurricane Electric BGP Toolkit
- Cloudflare Radar
- AppSec Santa
- Discord Engineering Blog
