# 🔍 Cybersecurity Incident Analysis: The 5-Tuple Containment

## 📝 Overview
This repository contains the **forensic analysis** and containment steps performed on a compromised host (209.165.200.235) based on alerts raised in **Security Onion**.

The analysis covers the complete **Kill Chain**: Initial Access (Root), Persistence (Backdoor), and Data Exfiltration (FTP).

## 🔑 Key Findings
- **Root Access:** Confirmed via `uid=0(root)`.
- **Persistence:** A root-level backdoor account (`myroot`) was injected into `/etc/passwd`.
- **Exfiltration:** Sensitive file (`confidential.txt`) was stolen via FTP using compromised credentials (`analyst:cyberops`).

---

## 📚 Documentation
- **[Analysis Report](ANALYSIS_REPORT.md)**: Detailed report with all findings, evidence links, and recommendations.
- **[Evidence Folder](Evidence/)**: Contains 10 highlighted screenshots from **Sguil**, **Wireshark**, and **Kibana**.

---

## 📜 License
This project is licensed under the **[MIT License](LICENSE)**.
