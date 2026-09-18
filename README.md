# NETWORKWALKS-B083-WEEK-2-PM1-CYBERSECURITY-FOOTPRINTING-WITH-MULTIPLE-TOOLS
Footprinting using six built-in Kali Linux tools: whois, whatweb, nslookup, curl, wafw00f and dnsrecon.

📌 Project Overview

This project focuses on **footprinting** — the process of gathering publicly available information about a target domain before any active testing takes place. Using six tools that ship with Kali Linux by default, I collected domain registration data, technology stack details, DNS records, and WAF (Web Application Firewall) information for a target domain.

Footprinting is entirely passive/low-noise reconnaissance: no exploitation is performed, only publicly accessible information is gathered.

 🎯 Objectives

- Gather domain registration (WHOIS) details for the target.
- Fingerprint the web technologies running on the target site.
- Resolve the domain's IP address via DNS lookup.
- Inspect HTTP response headers for server and caching information.
- Detect whether the target is protected by a Web Application Firewall.
- Enumerate DNS records (SOA, NS, MX, TXT, SRV) for the domain.
- Document commands, output, and key takeaways for each tool.

---

## 🛠️ Tools Used

| 🧰 Tool       | 🎯 Purpose                                             |
| ------------- | ------------------------------------------------------- |
| `whois`       | Domain registration details (registrar, dates, name servers) |
| `whatweb`     | Web technology fingerprinting (CMS, server, frameworks)  |
| `nslookup`    | DNS resolution — maps the domain to its IP address       |
| `curl -I`     | Retrieves raw HTTP response headers from the target       |
| `wafw00f`     | Detects the presence of a Web Application Firewall (WAF) |
| `dnsrecon`    | Broader DNS enumeration (SOA, NS, MX, TXT, SRV records)   |

**Note:** All lookups performed here are passive, publicly available OSINT (Open-Source Intelligence) queries against a domain used for authorized training purposes. No active exploitation or unauthorized access was attempted.


# 🪜 Footprinting Procedure & Findings

## 1. WHOIS Lookup

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

2. Web Technology Fingerprinting

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

## 3. DNS Resolution

```bash
nslookup networkwalks.com
```

**Findings:**
- Resolves to IP address `192.232.216.135`
- Queried via DNS server `5.11.11.5`

`nslookup` confirms the domain-to-IP mapping — a quick sanity check before running any tool that needs the target's actual IP.

 4. HTTP Header Inspection

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

 5. WAF Detection

```bash
wafw00f networkwalks.com
```

**Findings:**
- The site is protected by **ModSecurity (SpiderLabs) WAF**
- Detected in 2 requests

Knowing a WAF is in place ahead of time is important — it explains why certain scans or payloads might get blocked or rate-limited later, and shapes how testing would need to be approached in a real engagement.

6. DNS Enumeration

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

---

 💡 What I Learned

 1. Footprinting is Entirely Passive
Every tool used here queries publicly available information — no packets are sent that could be considered an attack. This makes footprinting a safe, legal first phase of any security assessment.

2. Multiple Tools, Overlapping but Different Data
`nslookup`, `dnsrecon`, and `whatweb` all touched on IP/DNS information, but each surfaced different details — reinforcing that a proper footprint uses several tools rather than relying on just one.

3. WAF Awareness Matters Early
Discovering a WAF in front of the target (via `wafw00f`) is valuable before any later scanning, since it explains unexpected blocks and shapes testing strategy.

4. Documentation Discipline
Recording the exact command, the raw output, and a short interpretation for each tool made the findings far easier to compare and reference later.

🔐 Security & Ethical Use

All footprinting activity in this project was performed against a domain used for authorized training purposes as part of the Networkwalks Cybersecurity program. These techniques should only be used against systems you own or have explicit written permission to test.
