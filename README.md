# Incident Response Report: HawkEye Keylogger Data Exfiltration

## 1. Executive Summary
An investigation into anomalous network activity revealed a successful malware infection and subsequent data exfiltration originating from an internal Windows endpoint. The threat actor compromised the host using a malicious payload identified as HawkEye Keylogger - Reborn v9. Following the infection, the malware successfully harvested sensitive credentials, including corporate banking access belonging to an accountant, and exfiltrated them via SMTP to an external server located in the United States.

## 2. Victim System Identification
The infected endpoint was identified as the most active computer at both the network and link levels during the capture period.

* **Local IP Address:** 10.4.10.132
* **MAC Address:** 00:08:02:1c:47:ae (Hewlett-Packard)
* **Operating System:** Windows
* **User Profile:** BEIJING-5CD1-PC\roman.mcguire

[Image: Screenshot of Wireshark endpoint statistics highlighting the victim IP and MAC address]

## 3. Malware Delivery & Indicators of Compromise (IoCs)
The infection occurred after the user (an accountant) downloaded a malicious executable file hosted on a remote web server.

* **Malicious Filename:** `tkraw Protected99.exe`
* **File Hash (MD5):** `71826BA081E303866CE2A2534491A2F7`
* **Distribution Domain:** `proforma-invoices.com`
* **Distribution IP Address:** 217.182.138.150
* **Hosting Web Server:** LiteSpeed

[Image: Screenshot of VirusTotal analysis showing the MD5 hash detection and threat classification]

## 4. Exfiltration Mechanism
Once established, the malware initiated a persistent data exfiltration routine, sending the stolen keylog and credential data out of the network via email.

* **Exfiltration Frequency:** Every 3 minutes
* **Destination Server:** 23.229.162.69 (Exim 4.91)
* **Threat Actor Email:** `sales.del@macwinlogistics.in`
* **Authentication Method:** The malware authenticated to the SMTP server using the Base64 encoded string `U2FsZXNAMjM=`, which decodes to the plaintext password `Sales@23`.

[Image: Screenshot of CyberChef decoding the Base64 SMTP authentication string]

## 5. Impact Assessment & Compromised Credentials
The keylogger successfully harvested several high-value account credentials associated with the user `roman.mcguire`. Immediate password resets and account security protocols must be initiated for the following compromised services:

| Service / Platform | Username / Email | Stolen Password | Capture Source |
| :--- | :--- | :--- | :--- |
| **Bank of America** | `roman.mcguire` | `P@ssword$` | Google Chrome |
| **AOL** | `roman.mcguire914@aol.com` | `P@ssword$` | Internet Explorer 7.0-9.0 |
| **Pizza Jukebox** | `roman.mcguire@pizzajukebox.com` | `P@ssw0rd$` | MS Outlook |

> **Note on Email Infrastructure:** The Pizza Jukebox email configuration was captured in the logs, revealing the POP3 server (`pop.pizzajukebox.com` on port 995) and the SMTP server (`smtp.pizzajukebox.com` on port 587).

[Image: Screenshot of the extracted PCAP TCP stream showing the plaintext Bank of America credentials being exfiltrated]

## 6. Root Cause Analysis (RCA) & Remediation Strategy
**Root Cause:**
The initial compromise occurred due to a failure in endpoint execution controls and user security awareness. The victim (an accountant) was able to successfully download and execute an untrusted payload (`tkraw Protected99.exe`) from an external domain (`proforma-invoices.com`) without being intercepted by endpoint antivirus or network web filters. 

**Recommended Remediation & Mitigation:**
To prevent recurring incidents and secure the network perimeter, the following tactical steps must be implemented:
* **Firewall Blacklisting:** Immediately block outbound traffic to the threat actor's SMTP server (`23.229.162.69`) and distribution IP (`217.182.138.150`).
* **Endpoint Detection & Response (EDR):** Deploy EDR solutions across all endpoints to automatically quarantine untrusted executables.
* **Principle of Least Privilege (PoLP):** Revoke local administrator rights from standard user accounts (like accounting) to prevent unauthorized software installation.
* **Security Awareness Training:** Mandate phishing and social engineering training for all finance and accounting personnel.
