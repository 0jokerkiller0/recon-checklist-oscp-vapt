# 🕵️‍♂️ OSCP & VAPT Information Gathering Checklist

**Target IP/URL:** `____________________`
**Date:** `YYYY-MM-DD`
**Scope:** [ ] Confirmed  [ ] Exclusions Noted

---

## 🟢 Phase 1: Passive Recon (OSINT)
*Goal: Gather info without touching the target directly. Essential for VAPT, useful for OSCP.*

- [ ] **Whois Lookup**
    - `whois <domain>` (Identify registrar, contacts)
- [ ] **DNS Enumeration**
    - `dig <domain> any`
    - `dig <domain> mx` (Mail servers)
    - `dig <domain> txt` (SPF records, verification tokens)
- [ ] **Subdomain Discovery** (If target is a domain)
    - `sublist3r -d <domain>`
    - `amass enum -d <domain>`
    - [CRT.sh](https://crt.sh/) check
- [ ] **Google Dorking**
    - `site:<domain> filetype:pdf`
    - `site:<domain> "password" OR "login"`
    - `site:<domain> intitle:"index of"`
- [ ] **Tech Stack Identification**
    - [ ] Wappalyzer (Browser Extension)
    - [ ] `whatweb <url>`

---

## 🟡 Phase 2: Active Network Scanning
*Goal: Identify live hosts and open ports.*

### 1. Host Discovery (Ping Sweep)
- [ ] **Internal/Lab:** `nmap -sn 192.168.X.0/24`
- [ ] **Real Life (Stealthier):** `netdiscover -r 192.168.X.0/24`

### 2. Port Scanning (The Nmap Trinity)
- [ ] **Initial Fast Scan (TCP):**
    - `nmap -sC -sV -oN nmap_initial <IP>`
- [ ] **Full Port Scan (TCP - Don't skip this!):**
    - `nmap -p- --min-rate=1000 -T4 -oN nmap_full <IP>`
- [ ] **UDP Scan (Top Ports):**
    - *Critical for SNMP (161) and TFTP (69)*
    - `nmap -sU --top-ports 50 -oN nmap_udp <IP>`

---

## 🔴 Phase 3: Service Enumeration
*Goal: Interrogate specific ports found in Phase 2.*

### 📂 FTP (Port 21)
- [ ] **Anonymous Login:** `ftp <IP>` -> User: `anonymous`, Pass: `anonymous`
- [ ] **Check Version:** Is it vsftpd 2.3.4? (Backdoor)
- [ ] **Download Everything:** `wget -m ftp://anonymous:anonymous@<IP>`

### 🔐 SSH (Port 22)
- [ ] **Banner Grab:** `nc -nv <IP> 22` (Check OS version/Distro)
- [ ] **Key Auth:** Do you have an `id_rsa` file from another service?
- [ ] **Weak Algo Check:** `ssh-audit <IP>` (For report writing)

### 📧 SMTP (Port 25/465/587)
- [ ] **User Enumeration:** `vrfy <username>`
- [ ] **Mail Relay:** Can you send email to an external address?

### 🌐 HTTP/HTTPS (Port 80/443/8080/8443)
- [ ] **Manual Browsing:**
    - [ ] View Source (`Ctrl+U`) - Look for comments/hidden inputs.
    - [ ] Check `/robots.txt` and `/sitemap.xml`.
- [ ] **Directory Busting:**
    - `gobuster dir -u <URL> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,sh`
    - *OR* `feroxbuster -u <URL>` (Recursive & Fast)
- [ ] **Vulnerability Scan:**
    - `nikto -h <URL>`
- [ ] **CMS Specific:**
    - **WordPress:** `wpscan --url <URL> -e p,t,u`
    - **Joomla:** `joomscan -u <URL>`
    - **Drupal:** `droopescan scan drupal -u <URL>`

### 📁 SMB (Port 139/445) - *High Value Target*
- [ ] **List Shares:** `smbclient -L //<IP> -N`
- [ ] **Map Shares:** `smbmap -H <IP>`
- [ ] **User Enum:** `enum4linux -a <IP>`
- [ ] **Check Vulns:** `nmap --script smb-vuln* -p 445 <IP>` (Detects EternalBlue)

### 📊 SNMP (Port 161/162 UDP)
- [ ] **Public String Check:** `snmpwalk -c public -v1 <IP>`
- [ ] **Enum Scripts:** `snmp-check <IP>`

### 🐘 Databases (3306 MySQL / 1433 MSSQL)
- [ ] **Default Creds:** root/root, admin/admin, sa/password
- [ ] **Empty Password:** Try logging in with no password.
- [ ] **Hydra Brute Force:** Only if authorized and necessary.

---

## 🟣 Phase 4: Mental Check & "The Rabbit Hole"
*Before moving to exploitation, ask yourself:*

- [ ] **Did I read the source code of every web page?**
- [ ] **Did I check the software versions for CVEs on Exploit-DB?**
- [ ] **Did I try default credentials on every login panel?**
- [ ] **Did I check for hidden files (Example: .git, .env, backup.zip)?**
- [ ] **Did I screenshot my scan results for the report?**

---
