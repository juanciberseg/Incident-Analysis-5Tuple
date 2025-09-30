# Forensic Incident Analysis Report: The 5-Tuple Containment

## Executive Summary  

A successful root-level compromise was detected on the host 209.165.200.235, originating from 209.165.201.17. The attacker achieved **privilege escalation**, established **persistence** by injecting a backdoor user (`myroot`), and successfully executed **data exfiltration** of a sensitive file (`confidential.txt`) via FTP.

Immediate containment was achieved by blocking the unique 5-Tuple associated with the connection, followed by remediation actions to remove persistence.

---

## 1. Initial Access and Privilege Escalation (Sguil & Wireshark)

The attack chain began with a successful exploit that granted the attacker root privileges on the compromised host.

### 1.1 Initial Alert and Root Confirmation

The security monitoring system (Sguil) generated an alert indicating a successful exploit response. Examination of the event transcript confirmed the execution of the `id` command, returning **`uid=0(root)`**.

* **Evidence [Capture 01]:** Sguil alert showing the initial exploit response.
* **Evidence [Capture 02]:** Sguil transcript confirming `uid=0(root)` and subsequent commands (`whoami`, `hostname`, `ifconfig`).

### 1.2 Host Reconnaissance

Following root access, the attacker performed reconnaissance commands to establish the host's configuration and search for sensitive information.

* **Evidence [Capture 03]:** Sguil transcript showing `ifconfig` output and the attacker's attempt to view the `/etc/shadow` file contents.
* **Evidence [Capture 05]:** Wireshark stream confirming the sequence of commands (`id`, `whoami`, `ifconfig`) and the initial attempts to set up persistence.

---

## 2. Persistence and Data Staging (Sguil & Wireshark)

The attacker focused on establishing a permanent backdoor and identifying data for exfiltration.

### 2.1 Backdoor Injection

The attacker injected a new user entry into the `/etc/passwd` file via an `echo` command to establish persistence using the username **`myroot`**.

* **Evidence [Capture 04]:** Sguil transcript showing the command `echo "myroot:..." >> /etc/shadow` and the subsequent verification command (`cat /etc/passwd`).

### 2.2 Credentials Disclosure

Further analysis of the raw stream traffic confirmed the attacker was able to successfully extract the hashed contents of the `/etc/shadow` file, gaining access to user credentials.

* **Evidence [Capture 06]:** Wireshark stream confirming the contents of the `/etc/shadow` file, including the compromised `root` and other user entries.

---

## 3. Data Exfiltration and Final Proof (Kibana)

The final stage involved the transfer of sensitive data using the FTP protocol, and the incident was tracked via Zeek and Kibana dashboards.

### 3.1 Exfiltration Details

The Zeek log data confirms the use of FTP credentials (**analyst** and **cyberops**) for a successful login and subsequent file transfer. The key command was **`STOR confidential.txt`**.

* **Evidence [Capture 10]:** Full Zeek log of the FTP session, showing the successful login, the `STOR confidential.txt` command, and the final **`226 Transfer complete`** response.

### 3.2 Kibana Dashboard Analysis

Kibana dashboards provide visual and summarized proof of the exfiltration event.

* **Evidence [Capture 07]:** Kibana's "Files" dashboard showing a single **`FTP_DATA`** file transfer event, confirming the use of FTP for data movement.
* **Evidence [Capture 09]:** Kibana's "FTP" dashboard confirming the **`STOR`** command count of **1**, and explicitly identifying the argument: **`ftp://209.165.200.235/confidential.txt`**.

* **Evidence [Capture 08]:** Zeek log showing the specific content of the transferred file, confirming its nature as **"CONFIDENTIAL DOCUMENT"**.

---

## 4. Containment, Eradication, and Remediation

### 4.1 Containment Action

The immediate containment was based on the confirmed 5-Tuple of the malicious connection:
- **Source IP:** 209.165.201.17
- **Source Port:** (Dynamic, but tied to the connection)
- **Destination IP:** 209.165.200.235 (Compromised Host)
- **Destination Port:** 6200 (Initial connection port)
- **Protocol:** TCP

A firewall rule was implemented to block all future connections matching this 5-Tuple pattern, effectively cutting off the attacker's remote shell access.

### 4.2 Eradication and Remediation Steps

1.  **Backdoor Removal:** The malicious user entry **`myroot`** was immediately deleted from `/etc/passwd` and `/etc/shadow` files to remove persistence.
2.  **Credential Reset:** All known or potentially compromised user passwords, including `root` and `analyst`, were immediately reset.
3.  **Vulnerability Patching:** The underlying vulnerability that allowed the initial root access (GPL ATTACK) was identified and patched.

