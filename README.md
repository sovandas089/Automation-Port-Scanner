# Parallel Network Port Scanner Suite

**Parallel TCP & UDP Nmap Scanner** — Enterprise-grade VAPT automation tools for comprehensive network service discovery and vulnerability assessment.

---

## 📋 Overview

This suite provides two specialized, high-performance network reconnaissance tools built on Nmap for security assessments:

- **TCP Port Scanner (v4.1)** — Full TCP SYN scanning with parallel multi-range execution, service/version enumeration, and integrated vulnerability detection
- **UDP Port Scanner (v2.1)** — Priority-based UDP scanning with protocol-specific probes for common services (DNS, SNMP, NTP, IKE)

Both tools execute MITRE ATT&CK technique **T1046 — Network Service Discovery** and are optimized for speed through:
- **4 parallel scanning jobs** (dividing 1-65535 port range into 16K-port chunks)
- **Real-time progress visualization** with spinner animations and ETA calculations
- **Structured output** with merged reports, XML exports, and service enumeration
- **Vulnerability detection** via Nmap NSE scripts (vuln, auth, default, banner)

---

## 🎯 Key Features

### TCP Port Scanner (v4.1)

**Phase 1: Full TCP SYN Scan**
- Scans all 65,535 TCP ports in parallel (4 jobs × 16K ports each)
- Rate-controlled: 300 packets/second, max-retries: 2
- Extracts clean open port list for Phase 2
- Real-time progress bar and ETA

**Phase 2: Deep Service & Vulnerability Detection**
- Service/version enumeration (`-sV -sC`)
- Version intensity: 9 (maximum accuracy)
- NSE scripts: `vuln,auth,default,banner`
- CVE extraction and reporting
- XML output for integration with CVSS scoring tools

**Output Files:**
- `part1-4.txt` — Raw TCP SYN results by port range
- `open_ports.txt` — Clean list of detected open ports
- `deep_scan.txt` — Full service enumeration and vulnerability hits
- `deep_scan.xml` — Machine-readable XML for scoring/import
- `FULL_SCAN_*.txt` — Merged markdown report

---

### UDP Port Scanner (v2.1)

**Phase 1: Priority UDP Ports (Fast)**
- Scans 28 critical UDP ports: DNS (53), SNMP (161), NTP (123), TFTP (69), IKE (500/4500), SSDP (1900), etc.
- Rate: 200 pkts/s, max-retries: 1
- Instant discovery of common services
- NSE probes: banner, dns-recursion, snmp-info, ike-version, ntp-info, tftp-enum, nbstat

**Phase 2: Full UDP Scan (All 65K Ports)**
- Parallel 4-job execution across complete port range
- Rate: 150 pkts/s (UDP is slower; respects host limits)
- Distinguishes between "open" and "open|filtered" states
- Firewall detection (open|filtered = ICMP unreachable blocked by WAF/IDS)

**Phase 3: Deep Service Detection**
- Version enumeration on confirmed open + open|filtered ports
- Version intensity: 7, max-retries: 2
- NSE scripts: banner, snmp-info, snmp-sysdescr, ike-version, ntp-info, ntp-monlist

**Phase 4: Protocol-Specific Probes** (automated post-processing)
- IKE/IPSec aggressive mode probing (UDP 500/4500)
- SNMP information enumeration (UDP 161)
- NTP monlist and clock skew detection (UDP 123)
- DNS recursion testing (UDP 53)

**Output Files:**
- `priority_udp.txt` — Fast results from Phase 1
- `udp_part1-4.txt` — Full UDP scan results by range
- `udp_open_ports.txt` — Confirmed open UDP ports
- `udp_open_filtered_ports.txt` — Firewall-blocked (open|filtered) ports
- `udp_deep_scan.txt` — Full service enumeration
- `ike_probe.txt`, `snmp_probe.txt`, `ntp_probe.txt`, `dns_probe.txt` — Protocol-specific results
- `UDP_FULL_SCAN_*.txt` — Merged report

---

## 🛠️ Installation & Prerequisites

### Requirements

**System:**
- Linux (Kali Linux, Parrot Security OS, or Ubuntu/Debian-based distro)
- **Root/sudo access** (required for TCP SYN scans and raw UDP sockets)
- Bash 4.0+

**Dependencies:**
```bash
# Install Nmap (if not already present)
sudo apt-get update && sudo apt-get install -y nmap

# Verify installation
nmap --version
```

**NSE Scripts (included with Nmap):**
Ensure Nmap is compiled with NSE scripting support:
```bash
nmap --script-help vuln | head -5  # Should display script help
```

### Setup

1. **Download the scripts:**
   ```bash
   wget https://your-repo/TCP_Port_Scanner -O tcp_scanner.sh
   wget https://your-repo/UDP_Port_Scanner -O udp_scanner.sh
   ```

2. **Make executable:**
   ```bash
   chmod +x tcp_scanner.sh udp_scanner.sh
   ```

3. **(Optional) Move to PATH for global access:**
   ```bash
   sudo mv tcp_scanner.sh /usr/local/bin/tcp-scanner
   sudo mv udp_scanner.sh /usr/local/bin/udp-scanner
   ```

---

## 📖 Usage

### TCP Port Scanner

**Basic syntax:**
```bash
sudo ./tcp_scanner.sh <target_ip>
```

**Examples:**

```bash
# Scan single host
sudo ./tcp_scanner.sh 192.168.1.100

# Interactive mode (prompt for IP)
sudo ./tcp_scanner.sh

# Scan entire subnet (requires root)
sudo ./tcp_scanner.sh 192.168.0.0/24  # Note: Requires Nmap CIDR support
```

**Output:**
```
[+] Target     : 192.168.1.100
[+] Output dir : tcp_scan_192_168_1_100_20240115_143022
[+] Started    : Wed Jan 15 14:30:22 PST 2024

[*] Phase 1 — Full TCP SYN scan (1-65535) — 4 parallel jobs
    Rate: 300 pkts/s | max-retries: 2 | ~16383 ports each

Job Progress:
Job 1 [████████░░░░░░░░░░] [scanning] 67% — ports 1-16383
Job 2 [██████████████░░░░░] [done    ] 100% — ports 16384-32767 (12 open ports)
Job 3 [████████████░░░░░░░] [scanning] 58% — ports 32768-49151
Job 4 [██████████░░░░░░░░░] [scanning] 45% — ports 49152-65535

Overall: [████████░░░░░░░░░░] 72% | Elapsed: 04m32s | ETA: 01m45s
```

---

### UDP Port Scanner

**Basic syntax:**
```bash
sudo ./udp_scanner.sh <target_ip>
```

**Examples:**

```bash
# Scan single host
sudo ./udp_scanner.sh 10.0.0.50

# Interactive mode
sudo ./udp_scanner.sh
```

**Output (Phase 1 - Fast):**
```
[+] Target     : 10.0.0.50
[+] Output dir : udp_scan_10_0_0_50_20240115_153044
[+] Started    : Wed Jan 15 15:30:44 PST 2024
[!] UDP scanning is slow — 4 parallel jobs reduce total time

[*] Phase 1 — Priority UDP ports (fast)
    28 ports: DNS,SNMP,IKE,NTP,TFTP,SSDP...

  ◐ Priority scan running... | 00m12s elapsed

  ✓ Priority scan done | 12s
  [!] Quick finds: 53/udp 123/udp 161/udp
```

---

## 📊 Output Files & Report Structure

### TCP Scanner Output Directory

```
tcp_scan_192_168_1_100_20240115_143022/
├── FULL_SCAN_192_168_1_100.txt    ← Main merged report (START HERE)
├── open_ports.txt                 ← Clean port list (CSV format)
├── deep_scan.txt                  ← Service/version enumeration
├── deep_scan.xml                  ← XML for CVSS scoring tools
├── part1.txt                       ← Raw results: ports 1-16383
├── part2.txt                       ← Raw results: ports 16384-32767
├── part3.txt                       ← Raw results: ports 32768-49151
├── part4.txt                       ← Raw results: ports 49152-65535
└── .progress/                      ← Internal progress logs (temporary)
    ├── log1.txt
    ├── log2.txt
    ├── log3.txt
    └── log4.txt
```

### UDP Scanner Output Directory

```
udp_scan_10_0_0_50_20240115_153044/
├── UDP_FULL_SCAN_10_0_0_50.txt    ← Main merged report (START HERE)
├── udp_open_ports.txt             ← Confirmed open ports
├── udp_open_filtered_ports.txt    ← Firewall-blocked (open|filtered)
├── priority_udp.txt               ← Phase 1 quick results
├── udp_part1.txt                  ← Raw results: ports 1-16383
├── udp_part2.txt                  ← Raw results: ports 16384-32767
├── udp_part3.txt                  ← Raw results: ports 32768-49151
├── udp_part4.txt                  ← Raw results: ports 49152-65535
├── udp_deep_scan.txt              ← Full service enumeration
├── ike_probe.txt                  ← IKE/IPSec aggressive probing
├── snmp_probe.txt                 ← SNMP enumeration (OID walk)
├── ntp_probe.txt                  ← NTP monlist / clock skew
├── dns_probe.txt                  ← DNS recursion testing
└── .progress/                      ← Internal progress logs (temporary)
```

---

## 🔍 Reading the Reports

### TCP Report Example

```
════════════════════════════════════════════════════════════════
  PARALLEL TCP NMAP SCAN REPORT
  Target      : 192.168.1.100
  Date        : Wed Jan 15 14:34:22 PST 2024
  Duration    : 7m 45s
  Tool        : parallel_tcp_nmap_v2.sh v4.1 (T1046)
════════════════════════════════════════════════════════════════

  Open TCP ports : 12
  Port list      : 22,80,443,3306,5432,8080,8443,9200,27017,5900,445,3389

[Phase 1: TCP SYN SCAN SUMMARY]
  22/tcp      open  ssh
  80/tcp      open  http
  443/tcp     open  https
  445/tcp     open  microsoft-ds
  3306/tcp    open  mysql
  ...

[Phase 2: SERVICE / VERSION / VULNERABILITY DETECTION]
  22/tcp  ssh     OpenSSH 7.4 (protocol 2.0)   [VULNERABLE: CVE-2018-15473]
  80/tcp  http    Apache httpd 2.4.6           [Slowloris DoS: CVE-2007-6750]
  443/tcp https   nginx 1.14.0                 [missing HSTS header]
  445/tcp smb     Samba 4.1.6                  [SMB signing not required]
  ...

!! CVEs DETECTED:
   ▸ CVE-2018-15473   (OpenSSH username enumeration)
   ▸ CVE-2007-6750    (Slowloris DoS)
   ▸ CVE-2024-12345   (nginx Range Header bypass)
```

### UDP Report Example

```
════════════════════════════════════════════════════════════════
  PARALLEL UDP NMAP SCAN REPORT
  Target      : 10.0.0.50
  Date        : Wed Jan 15 15:44:22 PST 2024
  Duration    : 18m 32s
════════════════════════════════════════════════════════════════

  UDP Port States:
  open          — service responded to probe
  open|filtered — no response (firewall blocking ICMP unreachable)
  closed        — ICMP port unreachable received

[PHASE 1 — PRIORITY UDP PORTS]
  53/udp   open   domain        DNS recursive
  123/udp  open   ntp           NTP v3/v4
  161/udp  open   snmp          SNMPv2c public/private accessible
  500/udp  open   isakmp        IKE aggressive mode enabled ⚠️

[PHASE 2 — FULL UDP SCAN SUMMARY (1-65535)]
  Confirmed open : 4 — 53,123,161,500
  Open|filtered  : 8 — 1194,5353,5683,...

[PHASE 3 — DEEP SERVICE DETECTION]
  53/udp   dns      BIND 9.9.5 (recursive queries allowed)
  123/udp  ntp      NTPv3 monlist enabled (NTP amplification attack possible)
  161/udp  snmp     SNMPv2c public community string found
  500/udp  isakmp   IKE Diffie-Hellman groups 1,2 (weak DH)

[PHASE 4 — PROTOCOL PROBES]
  [IKE/IPSec — UDP 500/4500]
    IKE Version 2.0 detected (fragmentation support enabled)
    Main mode & aggressive mode both supported
  
  [SNMP — UDP 161]
    SNMP Version: 2c (SNMP v3 not supported)
    System OID: 1.3.6.1.4.1.9.9.46.1.6.1.1.1.1
  
  [NTP — UDP 123]
    Version: NTP v3/v4 (mixed)
    Monlist: ENABLED (exploit vector: UDP amplification)
  
  [DNS — UDP 53]
    Recursion: ALLOWED (DNS amplification attack possible)
```

---

## 🚀 Advanced Usage & Integration

### Extracting Results for Further Analysis

**Get clean open port lists:**
```bash
# TCP ports
cat tcp_scan_192_168_1_100_*/open_ports.txt | tr '\n' ',' 

# UDP ports
cat udp_scan_10_0_0_50_*/udp_open_ports.txt | tr '\n' ','

# Combined port list for targeted Metasploit/Nessus scan
echo "192.168.1.100:$(cat tcp_scan_*_/open_ports.txt | tr '\n' ',')"
```

**Parse XML for CVSS v3.1 scoring:**
```bash
# Extract deep_scan.xml for import into vulnerability management tools
# (Tenable Nessus, Rapid7 InsightVM, etc.)
cat tcp_scan_*/deep_scan.xml | grep -oP 'cpe:/[^"]*' | sort -u
```

**Generate CSV reports:**
```bash
# Convert nmap output to CSV for spreadsheet analysis
# Using third-party tools like nmap2csv or custom parsers
grep -E "^[0-9]+/(tcp|udp)" tcp_scan_*/part*.txt | \
  awk -F'[:/]' '{print $1","$2","$3}' > ports_report.csv
```

### Scanning Multiple Targets

**Batch scan subnet:**
```bash
#!/bin/bash
SUBNET="192.168.1"
for i in {1..254}; do
    echo "[*] Scanning $SUBNET.$i..."
    sudo ./tcp_scanner.sh "$SUBNET.$i" &
done
wait
```

**Sequential with output aggregation:**
```bash
for target in 10.0.0.1 10.0.0.2 10.0.0.5; do
    sudo ./tcp_scanner.sh "$target"
    sudo ./udp_scanner.sh "$target"
    sleep 5  # Avoid host rate-limiting
done

# Merge all reports
cat tcp_scan_*/FULL_SCAN_*.txt > aggregated_tcp_report.txt
cat udp_scan_*/UDP_FULL_SCAN_*.txt > aggregated_udp_report.txt
```

---

## ⚙️ Tuning & Performance

### Adjusting Scan Speed

**For slower networks (high packet loss):**
- Open `tcp_scanner.sh` or `udp_scanner.sh`
- Change `--max-rate` from 300→200 (TCP) or 150→100 (UDP)
- Increase `--max-retries` from 2→3

**For faster networks (lab/internal):**
- Increase `--max-rate` to 500+ (TCP) or 200+ (UDP)
- Reduce `--max-retries` to 1

Example:
```bash
# Line ~120 in tcp_scanner.sh
--max-rate 500 --max-retries 1
```

### Parallel Job Optimization

Both tools hardcode 4 parallel jobs. To change:
```bash
# Edit line 67 (TCP) or line 50 (UDP)
RANGES=("1-8192" "8193-16384" "16385-24576" "24577-32767" "32768-41000" "41001-49151" "49152-57343" "57344-65535")

# Update loop: for i in 1 2 3 4 5 6 7 8; do
# Update wait: wait $PID1 $PID2 $PID3 $PID4 $PID5 $PID6 $PID7 $PID8
```

---

## 🔐 Security Considerations

- **Legal:** Only scan **authorized** systems (obtain written approval before testing)
- **Stealth:** These tools use aggressive scanning (SYN/UDP floods). For evasion scans, use Nmap directly with `-f` fragmentation
- **Detection:** IDS/IPS may flag high-rate scanning. Consider adding delays: `sleep 2` between job submissions
- **Credentials:** Store Nmap script arguments securely; avoid hardcoding SNMPv3 passwords
- **Audit Trail:** All output is timestamped and organized by directory; retain for compliance documentation

---

## 🐛 Troubleshooting

### "nmap: command not found"
```bash
sudo apt-get install nmap
```

### "Operation not permitted" / "This scan requires root"
```bash
# Ensure you use sudo (TCP SYN scans and raw UDP sockets require root)
sudo ./tcp_scanner.sh <target>
sudo ./udp_scanner.sh <target>
```

### Scans hang / no progress update
```bash
# Check if Nmap processes are running
ps aux | grep nmap

# Kill hanging processes if needed
sudo killall nmap

# Re-run the scan
```

### Very slow UDP scans
- UDP is inherently slower (no TCP 3-way handshake ACK confirmation)
- Common UDP ports may have rate-limiting; use Phase 1 first, then target specific ports
- Consider increasing `--max-retries` for `open|filtered` ports

### NSE scripts not executing
- Verify Nmap NSE support: `ls /usr/share/nmap/scripts/ | grep vuln`
- Update Nmap: `sudo apt-get install --only-upgrade nmap`
- Manually test: `sudo nmap --script vuln -p 22 <target>`

---

## 📝 Report Integration

### For VAPT Deliverables

1. **Copy XML output** (`deep_scan.xml`) to your CVSS scoring tool
2. **Extract CVE list** from merged report for risk scoring
3. **Map findings** to CWE/MITRE ATT&CK in your report framework
4. **Reference output directories** as appendix evidence in final deliverable

### Example CVSS v3.1 Scoring Integration

```bash
# Extract CVEs and score them
grep -oE 'CVE-[0-9]{4}-[0-9]+' tcp_scan_*/deep_scan.txt | sort -u > cves.txt

# Cross-reference with NVD API or local CVSS database
while read cve; do
    curl -s "https://services.nvd.nist.gov/rest/json/cves/1.0?keyword=$cve" | jq '.result.CVE_Items[0].impact.baseMetricV3.cvssV3_1.baseScore'
done < cves.txt
```

---

## 📚 References

- **MITRE ATT&CK:** [T1046 Network Service Discovery](https://attack.mitre.org/techniques/T1046/)
- **OWASP:** [Nmap Basics](https://owasp.org/www-community/attacks/Nmap)
- **Nmap Official:** [nmap.org](https://nmap.org)
- **CVE/CVSS:** [NVD - National Vulnerability Database](https://nvd.nist.gov)

---

## 📄 License & Disclaimer

**Author:** Cybersecurity Engineering  
**Version:** TCP v4.1 | UDP v2.1  
**Last Updated:** January 2024

**Disclaimer:** These tools are provided for authorized security testing only. Unauthorized network scanning is illegal in most jurisdictions. Always obtain written permission before testing any system.

---

## 🤝 Support & Feedback

For issues, feature requests, or improved versions:
1. Review Nmap documentation: `man nmap`
2. Test NSE scripts individually: `nmap --script-help <script_name>`
3. Consult OWASP testing guides for methodology alignment
4. Share findings via security research channels

---

**Happy scanning! 🎯**
