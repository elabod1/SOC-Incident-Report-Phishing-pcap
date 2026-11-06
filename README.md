# 🛡️ Network Forensic Analysis: Phishing & Data Exfiltration via HawkEye Keylogger

## 📌 Executive Summary
This repository contains a Network Forensic analysis of a raw PCAP file related to a phishing incident. An accountant received an email disguised as an invoice. Analysis of the PCAP file reveals the download of a malicious executable, lateral movement attempts, and successful data exfiltration via SMTP (HawkEye Keylogger). No screenshots are provided; all evidence is extracted directly from the PCAP file.

## 📊 Dataset Details
- **File:** `suspicious_traffic.pcap`
- **Analysis Tools:** Wireshark, Tshark (Command-Line)

## 🔍 Technical Investigation (Step-by-Step)

### 1. Initial Infection & Malware Download
By filtering the PCAP for HTTP traffic and DNS queries, the following malicious domain was identified:
`proforma-invoices.com` resolving to IP `217.182.138.150`.

A subsequent HTTP GET request was extracted:
```http
GET /proforma/tkraw_Protected99.exe HTTP/1.1
Host: proforma-invoices.com
```
The HTTP response contained a ~2 MB payload (TCP stream reassembly), which was identified as the malicious executable (tkraw_Protected99.exe).

2. Lateral Movement Attempts
The infected host (173.66.146.112) attempted to communicate with the internal Domain Controller PizzaJukebox-DC (10.4.10.4).
Filtering by SMB2 protocol (smb2) revealed requests attempting to access hidden shares (\\PizzaJukebox-DC\IPC$). The server responded with STATUS_OBJECT_NAME_NOT_FOUND, indicating the attempt was blocked.

3. Data Exfiltration (SMTP)
The most critical finding is the SMTP stream to a GoDaddy-hosted server (p3plcpnl0413.prod.phx3.secureserver.net).
Filtering by smtp or tcp.port == 587 revealed the exfiltration session:

Attacker Email: sales.del@macwinlogistics.in

Payload: Base64 encoded text containing stolen credentials from Bank of America, AOL, and the corporate Outlook account.

🚨 Indicators of Compromise (IOCs)
All extracted IOCs are documented in the iocs.txt file. Key indicators include:

Domain: proforma-invoices.com

IPs: 217.182.138.150, 23.229.162.69, 66.171.248.178

File: tkraw_Protected99.exe

🛡️ Recommendations
Block the identified IOCs at the Firewall and DNS levels.

Enforce Multi-Factor Authentication (MFA) for all corporate accounts.

Isolate and reimage the infected endpoint (173.66.146.112).

Review Domain Controller logs for any successful unauthorized access.

Disclaimer: This analysis was conducted for educational and incident response purposes.
