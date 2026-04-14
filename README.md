# Phishing Email Forensic Analysis

Forensic investigation of a cryptocurrency-themed phishing email (sample-996.eml) from the Phishing Pot dataset
## Overview
This project documents a hands-on phishing email analysis performed inside an isolated Ubuntu virtual machine. The investigation covers email header forensics, sender authentication evaluation (SPF, DKIM, DMARC), IP reputation analysis, and malicious URL dissection.
The sample email impersonates Binance to lure victims into a fake Arbitrum (ARB) token airdrop, directing them to attacker-controlled infrastructure via an obfuscated redirect chain.

###  Key Findings

Email Metadata

Sender (Display): BinanceMail
Sender (Actual): BinanceMail2@onmailcloud.onmicrosoft.com
To: phishing@pot@hotmail.com (honeypot address)
Subject: #rodrigofp: ARB Airdrop is now Live
Date: Wed, 26 Jul 2023, 18:32:33 +0000

### Sending Infrastructure

#### IP: 40.107.78.108
#### ISP: Microsoft Corporation
#### Hostname: mail-wa2pol01on2108.outbound.protection.outlook.com
#### Country: Poland 🇵🇱
#### AbuseIPDB: Not previously reported

### The attacker used a Microsoft-hosted tenant (onmailcloud.onmicrosoft.com) to pass SPF and evade IP reputation filters — a common technique to abuse trusted cloud infrastructure.
### Malicious URL

https://click.pstmrk.it/3s/sweedbuy.com%2Fblog%2F/ahc/k_CuAQ/AQ/
44a54f89-410d-4729-b21c-32c30d6eb945/1/qOoKiS9V1s?/23687658rodrigofp

The URL returned HTTP 404 at time of analysis — infrastructure was taken down after the campaign expired.

### Tools Used

#### Github — dataset acquisition
#### Mozilla Thunderbird — email rendering and visual inspection
#### Mousepad — raw .eml header analysis
#### AbuseIPDB — IP reputation lookup
#### Firefox — URL investigation

### Setup & Reproduction
bash# 1. Clone the Phishing Pot dataset
git clone https://github.com/rf-peixoto/phishing_pot.git

#### 2. Navigate to the email samples
cd phishing_pot/email

#### 3. Install Thunderbird (Ubuntu)
apt install thunderbird

#### 4. Open a sample in Thunderbird
thunderbird sample-996.eml

#### 5. Inspect raw headers in a text editor
mousepad sample-996.eml

⚠️ Always analyse phishing samples inside an isolated VM with no access to production networks or credentials.


📄 Full Report
The complete forensic report including detailed header breakdowns, routing analysis, and recommendations is available in report/Phishing_Analysis_Report.docx.

⚠️ Disclaimer
This repository is for educational and research purposes only. All analysis was performed in a controlled, isolated environment. The phishing sample is sourced from the publicly available Phishing Pot dataset. Do not use any of the techniques, IOCs, or infrastructure information described here for malicious purposes.

### References

#### Phishing Pot Dataset — rf-peixoto
#### AbuseIPDB — IP reputation database
#### MXToolbox Email Header Analyzer
#### RFC 7208 — SPF
#### RFC 6376 — DKIM
#### RFC 7489 — DMARC
