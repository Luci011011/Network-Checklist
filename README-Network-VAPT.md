# Network VAPT Checklist & Methodology

> A practical, command-oriented checklist for authorized Vulnerability Assessment and Penetration Testing (VAPT) of network infrastructure.

[![VAPT](https://img.shields.io/badge/Assessment-VAPT-blue)](#)
[![Network](https://img.shields.io/badge/Focus-Network-orange)](#)
[![Authorized Testing](https://img.shields.io/badge/Use-Authorized%20Testing-green)](#)

## Disclaimer

This repository is intended for **authorized security testing, defensive validation, labs, and education**. Run scans only against assets that you own or have explicit written permission to assess. Avoid denial-of-service, destructive exploitation, credential attacks, or disruptive scans unless they are explicitly approved in the rules of engagement (RoE).

---

## 1. Objectives

A network VAPT should identify and validate:

- Exposed hosts, ports, and services
- Outdated or vulnerable software
- Weak protocols and cryptographic configurations
- Insecure management interfaces
- Network segmentation and access-control weaknesses
- Misconfigured DNS, SMTP, SNMP, SMB, SSH, FTP, TLS, VPN, and web services
- Default or insecure configurations
- Excessive information disclosure
- Known CVEs and missing security patches
- Authentication and authorization weaknesses
- Monitoring and logging gaps

---

## 2. Recommended Methodology

```text
Authorization & Scope
        |
        v
Rules of Engagement
        |
        v
Asset Inventory / Recon
        |
        v
Host Discovery
        |
        v
TCP/UDP Port Discovery
        |
        v
Service & Version Enumeration
        |
        v
Configuration / TLS Review
        |
        v
Vulnerability Scanning
        |
        v
Manual Validation
        |
        v
Risk Rating + Evidence
        |
        v
Reporting
        |
        v
Remediation
        |
        v
Retest
        |
        v
Closure
```

This workflow follows the general planning, execution, analysis, reporting, and remediation principles described by NIST SP 800-115.

---

## 3. Pre-Assessment Checklist

- [ ] Written authorization received
- [ ] In-scope IP ranges documented
- [ ] In-scope hostnames documented
- [ ] Internal/external scope separated
- [ ] Production/staging scope identified
- [ ] Out-of-scope assets documented
- [ ] Testing window approved
- [ ] Emergency contact available
- [ ] Source IPs documented
- [ ] VPN/jump-host access confirmed
- [ ] Credentials supplied where applicable
- [ ] Scan intensity agreed
- [ ] DoS testing explicitly allowed or prohibited
- [ ] Social engineering excluded unless approved
- [ ] Data-handling requirements documented
- [ ] Evidence storage location agreed

### Scope template

| Item | Value |
|---|---|
| Client | `<organization>` |
| Internal ranges | `<CIDR>` |
| External IPs | `<IPs>` |
| Domains | `<domains>` |
| Test window | `<date/time>` |
| Tester source IP | `<IP>` |
| Test type | Black / Grey / White box |
| DoS testing | No / Controlled |
| Contact | `<contact>` |

---

# 4. Asset Discovery

## 4.1 Host discovery

```bash
nmap -sn 192.168.1.0/24 -oA discovery
```

Useful alternatives:

```bash
ping -c 3 <IP>
```

```bash
arp -a
```

### Checklist

- [ ] All expected hosts discovered
- [ ] Unexpected hosts identified
- [ ] Duplicate/stale assets investigated
- [ ] Asset ownership mapped

---

# 5. TCP Port Scanning

## 5.1 Full TCP SYN scan

```bash
sudo nmap -Pn -sS -p- --open <IP> -oA tcp-full
```

## 5.2 Service/version detection

```bash
sudo nmap -Pn -sS -sV --open <IP> -oA services
```

## 5.3 Common ports

```bash
nmap -Pn --top-ports 1000 -sV <IP> -oA top1000
```

## 5.4 Safe baseline scripts

```bash
nmap -Pn -sV -sC <IP> -oA baseline
```

### Checklist

- [ ] Open ports documented
- [ ] Service names documented
- [ ] Versions identified
- [ ] Unnecessary services identified
- [ ] Internet-exposed management services identified
- [ ] Unexpected ports investigated

---

# 6. UDP Assessment

Start conservatively because UDP scans can be slow.

```bash
sudo nmap -Pn -sU --top-ports 100 <IP> -oA udp-top100
```

For approved deeper testing:

```bash
sudo nmap -Pn -sU --top-ports 1000 <IP> -oA udp-top1000
```

Check especially:

- DNS/53
- SNMP/161
- NTP/123
- IPsec/IKE/500
- TFTP/69
- DHCP-related services

---

# 7. Service-Specific Checklist

## SSH — TCP/22

```bash
nmap -Pn -p22 --script ssh2-enum-algos <IP>
```

Check:

- [ ] SSHv1 disabled
- [ ] Weak algorithms disabled
- [ ] Weak MACs disabled
- [ ] Password authentication policy reviewed
- [ ] Root login policy reviewed
- [ ] MFA/strong authentication where required
- [ ] Internet exposure justified
- [ ] Version is supported

---

## TLS / SSL

```bash
nmap -Pn -p443 --script ssl-enum-ciphers <host>
```

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Using testssl.sh:

```bash
./testssl.sh https://example.com
```

Check:

- [ ] Certificate valid
- [ ] Correct hostname/SAN
- [ ] No expired certificate
- [ ] TLS 1.0/1.1 disabled where policy requires
- [ ] Weak ciphers disabled
- [ ] Strong key exchange
- [ ] Strong signature algorithm
- [ ] HSTS enabled where appropriate
- [ ] Certificate chain valid

---

## DNS — UDP/TCP 53

```bash
dig example.com
```

```bash
dig NS example.com
```

```bash
dig MX example.com
```

Authorized zone-transfer test:

```bash
dig AXFR @<authoritative-dns> example.com
```

Nmap:

```bash
nmap -Pn -sU -p53 --script dns-recursion,dns-service-discovery <IP>
```

Check:

- [ ] Unauthorized zone transfer prevented
- [ ] Recursion restricted
- [ ] DNS version disclosure minimized
- [ ] Internal records not unnecessarily exposed
- [ ] DNSSEC requirement evaluated

---

## SNMP — UDP/161

```bash
nmap -Pn -sU -p161 --script snmp-info <IP>
```

Check:

- [ ] SNMPv1/v2c usage reviewed
- [ ] Default community strings not used
- [ ] SNMP restricted to management networks
- [ ] SNMPv3 preferred where supported
- [ ] Read/write access appropriately restricted

---

## SMB — TCP/445

```bash
nmap -Pn -p445 --script smb-protocols,smb2-security-mode <IP>
```

Check:

- [ ] SMBv1 disabled
- [ ] Signing policy appropriate
- [ ] Guest/null access disabled
- [ ] Unnecessary shares removed
- [ ] Sensitive shares restricted
- [ ] Patch level reviewed
- [ ] External exposure prohibited unless justified

---

## FTP — TCP/21

```bash
nmap -Pn -p21 --script ftp-anon,ftp-syst <IP>
```

Check:

- [ ] Anonymous access disabled unless explicitly required
- [ ] Cleartext FTP avoided
- [ ] FTPS/SFTP preferred
- [ ] Chroot/jail configured
- [ ] Write permissions restricted

---

## SMTP — TCP/25, 465, 587

```bash
nmap -Pn -p25,465,587 --script smtp-commands <IP>
```

Check:

- [ ] Open relay prevented
- [ ] TLS configured appropriately
- [ ] Authentication required for submission
- [ ] Unnecessary commands disabled
- [ ] Banner disclosure minimized
- [ ] SPF/DKIM/DMARC reviewed for relevant domains

---

## HTTP/HTTPS

```bash
nmap -Pn -p80,443 --script http-title,http-headers <IP>
```

```bash
curl -I https://example.com
```

Check:

- [ ] HTTP redirects to HTTPS where required
- [ ] Security headers reviewed
- [ ] Server version disclosure minimized
- [ ] Directory listing disabled
- [ ] Admin interfaces restricted
- [ ] TLS configuration reviewed

---

## NTP — UDP/123

```bash
nmap -Pn -sU -p123 --script ntp-info <IP>
```

Check:

- [ ] NTP service required
- [ ] Unauthorized management restricted
- [ ] Version disclosure minimized
- [ ] Configuration follows organizational time-sync policy

---

## VPN / IPsec

```bash
sudo nmap -Pn -sU -p500,4500 <IP>
```

Where approved and appropriate:

```bash
ike-scan <IP>
```

Check:

- [ ] Strong authentication
- [ ] Strong cryptography
- [ ] Legacy protocols disabled
- [ ] Remote access restricted
- [ ] MFA enabled where supported
- [ ] Split-tunnel policy reviewed

---

# 8. Vulnerability Scanning

Use an approved vulnerability scanner such as:

- Nessus
- Greenbone/OpenVAS
- Qualys
- Rapid7 InsightVM

Before scanning:

- [ ] Scope imported correctly
- [ ] Credentials configured where authorized
- [ ] Plugins/feed updated
- [ ] Safe checks enabled
- [ ] DoS checks disabled unless explicitly approved
- [ ] Scan rate is appropriate
- [ ] Maintenance window confirmed

Nmap NSE can supplement scanners:

```bash
nmap -Pn --script vuln <IP>
```

**Important:** Scanner output is not automatically a confirmed vulnerability. Validate significant findings manually.

---

# 9. Configuration Review

Review:

- Firewall ACLs
- Router ACLs
- VLAN segmentation
- Management VLANs
- Internet-facing interfaces
- Default routes
- Unused interfaces
- Device management protocols
- Logging
- NTP
- SNMP
- AAA/RADIUS/TACACS+
- Backup configuration
- Configuration encryption
- Admin accounts
- Password policies
- Firmware/software versions

---

# 10. Network Segmentation Testing

With approved source and destination hosts:

```bash
nc -vz <target> <port>
```

```bash
nmap -Pn -p22,80,443,445 <target>
```

Validate that:

- [ ] User VLAN cannot reach restricted management VLAN
- [ ] Guest network is isolated
- [ ] Server-to-server access follows policy
- [ ] Database networks are restricted
- [ ] Management interfaces are not publicly reachable
- [ ] East-west traffic is appropriately filtered

---

# 11. Logging & Detection

Validate whether security controls detect:

- Port scanning
- Repeated authentication failures
- Access to restricted services
- Administrative login
- Configuration changes
- Suspicious DNS queries
- Unexpected management traffic

Record:

- Detection timestamp
- Source IP
- Destination
- Alert ID
- SOC response
- Logging source

---

# 12. Evidence Collection

For every finding capture:

- Asset/IP
- Port
- Service/version
- Command/tool
- Timestamp
- Raw output
- Screenshot where useful
- Business impact
- Remediation

Recommended directory:

```text
evidence/
├── discovery/
├── nmap/
├── tls/
├── dns/
├── smb/
├── vulnerability-scans/
├── screenshots/
└── reports/
```

Never commit credentials, private keys, API tokens, customer data, or confidential scan results to GitHub.

---

# 13. Severity

| Severity | Guidance |
|---|---|
| Critical | Direct or highly probable compromise with severe business impact |
| High | Significant unauthorized access, sensitive exposure, or exploitable weakness |
| Medium | Material security weakness requiring remediation |
| Low | Limited impact or defense-in-depth weakness |
| Informational | Hardening or observation |

Use CVSS v4.0 where appropriate.

---

# 14. Reporting Template

```text
Finding ID:
Title:
Asset:
IP/Hostname:
Port:
Protocol:
Severity:
CVSS:
CWE:
Description:
Evidence:
Steps to Validate:
Impact:
Recommendation:
References:
Retest Status:
```

---

# 15. Retesting

```text
Finding
   ↓
Remediation
   ↓
Repeat Original Test
   ↓
Verify Security Control
   ↓
Check Regression
   ↓
Pass / Fail
```

---

# 16. Toolset

### Discovery
- Nmap
- Amass
- dig
- nslookup
- arp

### Assessment
- Nessus
- Greenbone/OpenVAS
- Qualys
- Nmap NSE

### Protocol testing
- OpenSSL
- testssl.sh
- Wireshark
- Netcat
- SNMP utilities
- ike-scan

### Documentation
- Markdown
- Screenshots
- CSV/JSON exports
- CVSS calculator

---

# 17. References

- NIST SP 800-115: https://csrc.nist.gov/pubs/sp/800/115/final
- OWASP WSTG: https://owasp.org/www-project-web-security-testing-guide/
- Reference network checklist: https://github.com/qaisarafridi/-Network-Security-VAPT-Checklist-Methodology

---

## License

Use an appropriate license for your own repository. Do not copy third-party repository content verbatim without checking its license and attribution requirements.
