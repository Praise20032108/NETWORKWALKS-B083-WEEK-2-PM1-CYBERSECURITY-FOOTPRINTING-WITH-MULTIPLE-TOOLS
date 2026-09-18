# Cybersecurity — Week 2: Reconnaissance Labs

**Program:** Cybersecurity at Networkwalks
**Batch:** B083 | **Week:** 2
**Labs covered:**
PM1 – Footprinting with Multiple Tools 
PM5 – Network Scanning with Zenmap

## 📌 Overview

This README combines two related reconnaissance labs from Week 2. Both labs sit in the same phase of a security assessment — gathering information before any active testing — but they operate at different layers:

- **Project 1 (Footprinting)** gathers *publicly available* information about an external domain (WHOIS, DNS, web stack, WAF) using passive OSINT tools.
- **Project 2 (Network Scanning with Zenmap)** performs *active discovery* on a local/private network to identify which hosts are alive, using Zenmap (the Nmap GUI).

Together they illustrate the two common starting points of recon: passive footprinting of an internet-facing target, and active host discovery on a network you control.

## 📑 Table of Contents

1. [Project 1: Footprinting with Multiple Tools](#project-1-footprinting-with-multiple-tools)
2. [Project 2: Network Scanning with Zenmap](#project-2-network-scanning-with-zenmap)
3. [Combined Takeaways](#-combined-takeaways)
4. [Security & Ethical Use](#-security--ethical-use)

---

## Project 1: Footprinting with Multiple Tools

Footprinting using six built-in Kali Linux tools: `whois`, `whatweb`, `nslookup`, `curl`, `wafw00f`, and `dnsrecon`.

### 🎯 Objectives

- Gather domain registration (WHOIS) details for the target.
- Fingerprint the web technologies running on the target site.
- Resolve the domain's IP address via DNS lookup.
- Inspect HTTP response headers for server and caching information.
- Detect whether the target is protected by a Web Application Firewall.
- Enumerate DNS records (SOA, NS, MX, TXT, SRV) for the domain.
- Document commands, output, and key takeaways for each tool.

### 🛠️ Tools Used

| 🧰 Tool | 🎯 Purpose |
|---|---|
| `whois` | Domain registration details (registrar, dates, name servers) |
| `whatweb` | Web technology fingerprinting (CMS, server, frameworks) |
| `nslookup` | DNS resolution — maps the domain to its IP address |
| `curl -I` | Retrieves raw HTTP response headers from the target |
| `wafw00f` | Detects the presence of a Web Application Firewall (WAF) |
| `dnsrecon` | Broader DNS enumeration (SOA, NS, MX, TXT, SRV records) |

> **Note:** All lookups performed here are passive, publicly available OSINT queries against a domain used for authorized training purposes. No active exploitation or unauthorized access was attempted.

### 🪜 Procedure & Findings

#### 1. WHOIS Lookup

```bash
whois networkwalks.com
```

**Findings:**
- Registrar: GoDaddy.com, LLC
- Domain created: 2019-11-06
- Registry expiry date: 2027-11-06
- Name servers: `NS6135.HOSTGATOR.COM`, `NS6136.HOSTGATOR.COM`
- DNSSEC: Unsigned
- Domain locked against transfer/update/delete/renew (client-side EPP statuses)

WHOIS reveals who owns a domain, when it was registered, and which name servers manage its DNS — useful as a starting point for understanding a target's infrastructure and hosting provider.

#### 2. Web Technology Fingerprinting

```bash
whatweb networkwalks.com
```

**Findings:**
- Site redirects from HTTP to HTTPS (301 Moved Permanently)
- Running **Apache** on **WordPress 7.1**
- Front-end built with **Bootstrap 7.1** and **jQuery 3.7.1**
- Hosted at IP `192.232.216.135`, geolocated to the United States
- Page title: "Networkwalks Academy"

`whatweb` fingerprints the software stack behind a website, which helps identify known vulnerabilities tied to specific CMS or plugin versions later in a real assessment.

#### 3. DNS Resolution

```bash
nslookup networkwalks.com
```

**Findings:**
- Resolves to IP address `192.232.216.135`
- Queried via DNS server `5.11.11.5`

`nslookup` confirms the domain-to-IP mapping — a quick sanity check before running any tool that needs the target's actual IP.

#### 4. HTTP Header Inspection

```bash
curl -I https://networkwalks.com
```

**Findings:**
- Response: `HTTP/2 200`
- Server: Apache
- `x-nginx-cache` header present, indicating an Nginx caching layer in front of Apache
- WordPress-related cookie (`__wpdm_client`) set with `Secure` and `HttpOnly` flags
- Content-Security-related `permissions-policy` header referencing Google reCAPTCHA/Cloudflare challenge domains

Raw HTTP headers often reveal server software, caching layers, and security headers (or the lack of them) without needing to load the full page.

#### 5. WAF Detection

```bash
wafw00f networkwalks.com
```

**Findings:**
- The site is protected by **ModSecurity (SpiderLabs) WAF**
- Detected in 2 requests

Knowing a WAF is in place ahead of time is important — it explains why certain scans or payloads might get blocked or rate-limited later, and shapes how testing would need to be approached in a real engagement.

#### 6. DNS Enumeration

```bash
dnsrecon -d networkwalks.com
```

**Findings:**
- SOA: `ns6135.hostgator.com` (50.87.144.87)
- NS records: `ns6135.hostgator.com`, `ns6136.hostgator.com` (both allow recursion — flagged as a warning)
- MX record: `mail.networkwalks.com` → `192.232.216.135`
- TXT records: SPF record (`v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`) and a Google site-verification token
- SRV records: 6 `_autodiscover._tcp` entries pointing to `cpanelemaildiscovery.cpanel.net` across multiple IPs
- Total: 8 DNS records found

`dnsrecon` gives a much fuller picture of a domain's DNS footprint than a single `nslookup` — mail servers, SPF policy, and autodiscover records can reveal hosting/email providers and additional attack surface.

### 💡 What I Learned (Project 1)

1. **Footprinting is entirely passive** — every tool used here queries publicly available information; no packets are sent that could be considered an attack. This makes footprinting a safe, legal first phase of any security assessment.
2. **Multiple tools, overlapping but different data** — `nslookup`, `dnsrecon`, and `whatweb` all touched on IP/DNS information, but each surfaced different details, reinforcing that a proper footprint uses several tools rather than relying on just one.
3. **WAF awareness matters early** — discovering a WAF in front of the target (via `wafw00f`) is valuable before any later scanning, since it explains unexpected blocks and shapes testing strategy.
4. **Documentation discipline** — recording the exact command, the raw output, and a short interpretation for each tool made the findings far easier to compare and reference later.

---

## Project 2: Network Scanning with Zenmap

Discovering live hosts on a local network using Zenmap (the Nmap GUI).

### 🎯 Objectives

- Review the host machine's network adapters and IP configuration.
- Identify a target IP address on the local network to scan.
- Run a ping scan in Zenmap to check if the target host is alive.
- Interpret the scan output and understand what a ping scan does (and doesn't) reveal.
- View the scan result as a network topology diagram.
- Understand Zenmap's topology legend (host status icons, colors, and connection types).

### 🛠️ Tools Used

| 🧰 Tool | 🎯 Purpose |
|---|---|
| `ipconfig /all` | Reviews the host's network adapters and assigned IPs |
| Zenmap | GUI front-end for Nmap — used to configure and run scans |
| Nmap (`nmap -sn`) | Underlying scan engine; `-sn` performs a host-discovery / ping scan without port scanning |

### ⚙️ Environment

| 🧩 Component | ⚙️ Details |
|---|---|
| Host OS | Windows |
| Relevant adapter | Ethernet 2 (VirtualBox Host-Only Network) — `192.168.56.1/24` |
| Wi-Fi adapter | `10.201.40.243/24` (separate network, not scanned) |
| Scan target | `192.168.56.1` |
| Scan type | Ping scan (`nmap -sn`) |

### 🪜 Procedure & Findings

#### 1. Review Network Configuration

```cmd
ipconfig /all
```

Running `ipconfig` on the host first confirmed which network adapters were active and what IP ranges were in use. The relevant adapter for this scan was **Ethernet 2**, a VirtualBox Host-Only network adapter assigned `192.168.56.1` with subnet mask `255.255.255.0`. Other adapters (main Ethernet, most Wireless LAN adapters, Bluetooth Network Connection) showed as disconnected, and the Wi-Fi adapter was on a separate `10.201.40.x` network not used for this scan.

#### 2. Run a Ping Scan in Zenmap

**Target:** `192.168.56.1`
**Profile:** Ping scan
**Command generated by Zenmap:**

```bash
nmap -sn 192.168.56.1
```

**Output:**

```
Starting Nmap 7.991 ( https://nmap.org ) at 2026-09-18 14:04 +0200
Nmap scan report for 192.168.56.1
Host is up.
Nmap done: 1 IP address (1 host up) scanned in 0.99 seconds
```

A ping scan (`-sn`) simply checks whether a host is up — it does not scan ports or attempt to identify running services. It's typically the first step in reconnaissance: confirm a host is alive before investing time in a deeper scan.

#### 3. Visualize the Scan with the Topology Viewer

Zenmap's **Topology** tab plotted the scanned host as a node connected to `localhost`, giving a simple visual map of the one live host found on the `192.168.56.1` network.

#### 4. Review the Topology Legend

The Topology Legend clarifies how to read the diagram:

- **Circle color** indicates how many open ports a host has (green = fewer than 3, yellow = 3–6, red = more than 6).
- **Square icons with green/yellow/red** mark a host as a router, switch, or wireless access point.
- **Line style** between hosts shows traceroute information — solid for the primary path, dashed for missing or no traceroute data, and line thickness reflects round-trip time.
- Additional icons distinguish routers, switches, wireless access points, firewalls, and hosts with filtered ports.

Since this was a ping scan only (no port scan), the host appeared without port-count coloring, since it hadn't been port-scanned yet.

### 💡 What I Learned (Project 2)

1. **Ping scans are a discovery step, not a full scan** — `nmap -sn` only tells you whether a host is alive; it's a fast, low-noise way to build a list of live targets before running heavier scans (e.g., port or service scans) against them.
2. **Zenmap translates GUI choices into real Nmap commands** — selecting a scan profile (like "Ping scan") in Zenmap automatically builds the equivalent Nmap command line, a useful way to learn Nmap syntax while still using a visual interface.
3. **Knowing your own network config first matters** — running `ipconfig` before scanning made it clear which adapter and IP range to target, avoiding wasted scans against the wrong network (e.g., the Wi-Fi network instead of the VirtualBox host-only network).
4. **The topology view adds context** — the topology diagram and its legend make it easier to interpret scan results at a glance, especially once more hosts and open ports are involved in later, more advanced scans.

---

## 💡 Combined Takeaways

- Recon splits naturally into **passive** (footprinting — WHOIS, DNS, HTTP headers, WAF detection) and **active** (host discovery — ping scans) phases, and both are typically completed before any exploitation is attempted.
- No single tool tells the whole story: each lab used multiple tools that overlapped in places (e.g., DNS resolution appearing in both `nslookup`/`dnsrecon` and, conceptually, `ipconfig` for local addressing) but each contributed unique details.
- Understanding your own environment first — whether that's your local network adapters or the target's DNS/hosting setup — prevents wasted effort and wrong-target mistakes.
- Visualizing results (Zenmap's topology view) and documenting commands/output/interpretation (footprinting write-up) both make findings far easier to review and build on later.

## 🔐 Security & Ethical Use

- The **footprinting** activity was performed against a domain used for authorized training purposes as part of the Networkwalks Cybersecurity program.
- The **network scan** was performed against a host-only virtual network adapter under my own control, strictly for learning purposes.

These techniques should only ever be used against systems and networks you own or have explicit written permission to test.

---

**Program:** Cybersecurity at Networkwalks | **Projects:** Footprinting with Multiple Tools · Network Scanning with Zenmap
