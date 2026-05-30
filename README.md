# Phishing Email Analysis Report — Forensic Investigation of sample-996.eml

**Analyst:** Rukayat Mopelola Lawal  
**Analysis Date:** April 13, 2026  
**Dataset:** Phishing Pot (`rf-peixoto/phishing_pot`)  
**Sample:** sample-996.eml

---

## Overview

A forensic investigation of a phishing email sourced from the open-source Phishing Pot dataset. The email was analysed within an isolated Ubuntu virtual machine using Mozilla Thunderbird for rendering and Mousepad for raw header inspection. The investigation covered email authentication mechanisms (SPF, DKIM, DMARC), sender infrastructure, header routing, and embedded malicious URLs.

The email impersonates Binance to lure recipients into a fraudulent Arbitrum (ARB) token airdrop, using social engineering, urgency cues, and a spoofed sender identity to harvest credentials or redirect victims to attacker-controlled infrastructure.

---

## Analysis Environment

| Component | Details |
|-----------|---------|
| VM Platform | Oracle VirtualBox |
| Guest OS | Ubuntu 24 LTS (Clean State) |
| Email Client | Mozilla Thunderbird 140.9.1esr |
| Text Editor | Mousepad (raw .eml inspection) |
| IP Reputation Tool | AbuseIPDB (abuseipdb.com) |

---

## Dataset

The phishing corpus was cloned from the public GitHub repository:

```
git clone https://github.com/rf-peixoto/phishing_pot.git
```

- **Repository:** https://github.com/rf-peixoto/phishing_pot
- **Total objects:** 9,385 (130.66 MiB)
- **Target file:** `~/phishing_pot/email/sample-996.eml`

---

## Email Metadata

| Field | Value |
|-------|-------|
| From (Display) | BinanceMail |
| From (Address) | BinanceMail2@onmailcloud.onmicrosoft.com |
| To | "phishing@pot" \<phishing@pot@hotmail.com\> |
| Subject | #rodrigofp: ARB Airdrop is now Live |
| Date | Wed, 26 Jul 2023, 18:32:33 +0000 |
| Message-ID | WA0P291MB008013089869A243AC63A626C000A@WA0P291MB0080.POLP291.PROD.OUTLOOK.COM |
| Accept-Language | en-US |

---

## Social Engineering Techniques

The email body impersonated Binance and employed the following tactics:

- **Urgency framing:** Limited-time offer expiring July 29, 2023 at 18:00 UTC
- **Financial incentive:** First 2,500 applicants could receive up to 80,000 ARB (~USD $80,000)
- **Scarcity claim:** 50,000,000 ARB tokens on a first-come, first-served basis
- **Call to action:** Prominent yellow "Join Airdrop" button linking to a malicious URL
- **Impersonation:** Signed off as "Binance.com" with official-looking branding

---

## Email Authentication Analysis

### SPF

| Check | Result |
|-------|--------|
| SPF Result | **PASS** |
| Sender IP | 40.107.78.108 |
| smtp.mailfrom | onmailcloud.onmicrosoft.com |
| Notes | Attacker used a legitimate Microsoft-hosted tenant to pass SPF; bypasses IP reputation filters |

### DKIM

| Header Section | DKIM Result |
|----------------|-------------|
| Authentication-Results-Original | dkim=none (message not signed) |
| ARC-Authentication-Results i=1 | dkim=pass; header.d=onmailcloud.onmicrosoft.com |
| ARC-Authentication-Results i=2 | dkim=none (message not signed) |
| Overall Assessment | Unsigned at origin; ARC chain added by Microsoft relay only |

> The absence of DKIM signing means the email body and headers cannot be verified as unaltered — a significant red flag.

### DMARC

| Attribute | Value |
|-----------|-------|
| DMARC Result (Original) | dmarc=none action=none |
| Effective Policy | bestguesspass (no explicit DMARC policy published) |
| Risk | No enforced reject/quarantine policy on sender domain |

> No published DMARC policy means no enforcement action would be triggered even if authentication failed — characteristic of disposable Microsoft tenant domains used by threat actors.

---

## Sender IP Analysis

| Attribute | Value |
|-----------|-------|
| Sending IP | 40.107.78.108 |
| ISP | Microsoft Corporation |
| Usage Type | Data Center / Web Hosting / Transit |
| Hostname | mail-wa2pol01on2108.outbound.protection.outlook.com |
| Country | Poland (Warsaw, Mazovia) |
| AbuseIPDB Status | Not found in database |

The IP belongs to Microsoft's outbound Office 365 relay infrastructure, confirming the attacker abused a Microsoft-hosted tenant to exploit trusted sending infrastructure.

---

## Mail Routing

| Hop | Server | Timestamp |
|-----|--------|-----------|
| 1 (Origin) | WA0P291MB0080.POLP291.PROD.OUTLOOK.COM | 26 Jul 2023 18:32:50 +0000 |
| 2 | WA2P291MB0020.POLP291.PROD.OUTLOOK.COM | 26 Jul 2023 18:32:50 +0000 |
| 3 | DM6NAM12FT090.eop-nam12.prod.protection.outlook.com | 26 Jul 2023 18:32:57 +0000 |
| 4 | DS7PR03CA0259.outlook.office365.com | 26 Jul 2023 18:32:57 +0000 |
| 5 (Delivery) | MN0PR19MB5729.namprd19.prod.outlook.com | 26 Jul 2023 18:32:58 +0000 |

---

## Malicious URL Analysis

The "Join Airdrop" button contained the following embedded URL:

```
https://click.pstmrk.it/3s/sweedbuy.com%2Fblog%2F/ahc/k_CuAQ/AQ/44a54f89-410d-4729-b21c-32c30d6eb945/1/qOoKiS9V1s?/23687658rodrigofp
```

| Component | Value | Notes |
|-----------|-------|-------|
| Tracker Domain | click.pstmrk.it | Postmark click-tracking — abused for URL obfuscation |
| Destination Domain | sweedbuy.com | Fraudulent/spoofed crypto platform |
| Tracking ID | 44a54f89-410d-4729-b21c-32c30d6eb945 | Per-victim unique GUID |
| Campaign Tag | 23687658rodrigofp | Attacker campaign/affiliate identifier |

> At time of analysis (April 2026), the URL returned HTTP 404 — phishing infrastructure taken offline after the July 2023 campaign expired.

---

## Indicators of Compromise (IOCs)

| Type | Value | Confidence |
|------|-------|------------|
| Sender Email | BinanceMail2@onmailcloud.onmicrosoft.com | High |
| Sending IP | 40.107.78.108 | High |
| Return-Path Domain | onmailcloud.onmicrosoft.com | High |
| Redirect URL | click.pstmrk.it/3s/sweedbuy.com%2Fblog%2F/... | High |
| Destination Domain | sweedbuy.com | High |
| Campaign Tag | 23687658rodrigofp | Medium |
| Victim GUID | 44a54f89-410d-4729-b21c-32c30d6eb945 | Medium |
| Subject Pattern | #rodrigofp: ARB Airdrop is now Live | Medium |
| Message-ID Domain | WA0P291MB0080.POLP291.PROD.OUTLOOK.COM | Low |

---

## Key Evasion Techniques

- **Trusted Infrastructure Abuse:** Sending via Microsoft-hosted tenant to pass SPF and avoid IP reputation blocks
- **Click-Tracking Obfuscation:** Using Postmark's legitimate tracking domain to conceal the malicious destination URL
- **No DKIM Signing:** Reduces forensic traceability to a specific signing key
- **No DMARC Policy:** Tenant domain had no enforcement policy, allowing the email through unchallenged
- **Social Engineering:** Cryptocurrency airdrop theme exploiting FOMO and financial greed
- **Urgency & Scarcity:** Hard deadline and limited supply claims to pressure immediate action

---

## Authentication Summary

| Check | Result | Risk Level |
|-------|--------|------------|
| SPF | PASS (via Microsoft tenant) | Medium — passes due to cloud abuse |
| DKIM | NONE (not signed) | High — cannot verify message integrity |
| DMARC | NONE (no policy published) | High — no enforcement action possible |
| ARC | Pass (relay chain only) | Low — added by relay, not sender |
| IP Reputation | Clean (Microsoft infrastructure) | Medium — trusted IP misused |
| URL Safety | Malicious (inactive at analysis) | Critical |

---

## Recommendations

### For Email Security Teams

- Enforce DMARC with `reject` or `quarantine` policy on all organisational domains
- Implement DKIM signing on all outbound mail for cryptographic message integrity
- Deploy URL sandboxing and click-time URL rewriting to detect obfuscated redirect chains
- Configure mail gateway rules to flag `onmicrosoft.com` tenant emails impersonating financial institutions
- Subscribe to threat intelligence feeds covering newly registered or disposable Microsoft tenants

### For End Users

- Always verify the actual sender address, not just the display name — "BinanceMail" is not binance.com
- Be sceptical of unsolicited cryptocurrency airdrop or financial reward emails
- Do not click links in emails — navigate directly to official websites instead
- Report suspicious emails to the security team or flag them as phishing in your email client

---

## Conclusion

The analysis confirms sample-996.eml as a phishing email employing cryptocurrency-themed social engineering to deceive victims into visiting a malicious web portal. The attacker demonstrated a clear understanding of email authentication mechanisms — deliberately using a Microsoft-hosted tenant to pass SPF while exploiting the absence of DKIM and DMARC enforcement. The use of Postmark's legitimate click-tracking service to obfuscate the destination URL highlights the evolving sophistication of phishing infrastructure. This case reinforces the necessity of layered email security controls, particularly DMARC enforcement and URL inspection, alongside continuous user awareness training.
