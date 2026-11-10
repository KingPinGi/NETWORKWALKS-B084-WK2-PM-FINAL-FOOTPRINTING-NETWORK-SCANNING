# PENETRATION TESTING REPORT (WEEK 2)
### Footprinting, Reconnaissance & Network Scanning

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| Field | Detail |
|---|---|
| **Pentester Name** | **Oluwatobiloba Aina** |
| **Program / Batch** | B084, Networkwalks Cybersecurity Internship |
| **Date** | 10 November 2026 |
| **Modules covered** | W2-PM1: Footprinting with Multiple Kali Tools<br>W2-PM2: Footprinting with Google Hacking Database (GHDB)<br>W2-PM4: Passive Enumeration with theHarvester<br>W2-PM5: Network Scanning with Zenmap |
| **Client / Target** | 1. networkwalks.com, the authorized lab target of the NetworkWalks program<br>2. My own local LAN network, my own devices |
| **Permission secured?** | Yes, written authorization as part of the assigned program lab, plus self-owned infrastructure |
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting<br>**Phase 2:** Scanning & Network Discovery<br>**Phase 3-5:** In progress |

# 1. Liability Disclaimer

Every activity documented in this report was performed only against the authorized lab target of the NetworkWalks program, or against devices and networks that belong to me. The exposed systems found through Google dorks (GHDB) are third-party devices located through publicly indexed search results; they were observed only as lab evidence and were never accessed, altered or tested.

This report is for education and research purposes only. Do not use anything in it against systems you do not own or do not have written permission to test. Unauthorized access is a crime in most countries even when nothing is damaged. Every action you take is your own responsibility.

# 2. Introduction

Week 2 of my NetworkWalks internship moved from lab setup into the first two phases of ethical hacking: reconnaissance and scanning. This report documents four modules of practical work: footprinting a live authorized domain with six Kali Linux tools, discovering accidentally exposed public information with Google dorks, passively harvesting emails and sub-domains with theHarvester, and scanning my own local network with Zenmap.

Together, the four modules show the full journey that an attacker's preparation takes: starting from reading what is already public, and ending with a map of live hosts on a network, all without sending a single malicious packet at the target.

Every activity below includes the command used, what I observed, why the finding matters from an attacker's point of view, and how a defender should respond.

# 3. Tools Used

| Tool | Purpose |
|---|---|
| WHOIS | Domain registration details (owner, dates, name servers) |
| WhatWeb | Web technology fingerprinting (server, CMS, plugins, IP) |
| nslookup | DNS resolution of the domain to its IP address |
| curl -I | HTTP response headers of the website |
| wafw00f | Web Application Firewall detection |
| dnsrecon | Full DNS record enumeration (NS, MX, SPF, TXT, SRV) |
| GHDB / Google dorks | Finding accidentally exposed information via search operators |
| theHarvester | Passive email, sub-domain and host enumeration from public sources |
| Zenmap (Nmap GUI) | Local network host discovery (ping scan) and topology mapping |
| Windows CMD (ipconfig) | Local IP address and subnet identification |

# 4. Activities Performed

## 4.1 Footprinting with Multiple Kali Tools (W2-PM1)

I footprinted the authorized target networkwalks.com using six Kali Linux tools, with each tool collecting a different layer of public information.

**1. WHOIS.** I queried the public domain registration record and obtained the registrar, the registration and expiry dates, and the name servers. The name servers point to **HostGator**, so the hosting provider is identified instantly. Registration dates and abuse contacts are useful material for social engineering and attack planning.

**2. WhatWeb.** I fingerprinted the web stack. The tool identified **WordPress 7.0.4** and the **WP Download Manager 3.3.58** plugin, along with other details the site exposes, including an email address. An attacker looks these exact versions up in vulnerability databases to find known exploits.

**3. nslookup.** I resolved the domain name to its IP address: **192.232.216.135**. Knowing the IP lets an attacker scan the server directly, check for other sites on the same host, and map the target's hosting infrastructure.

**4. curl -I.** I read the HTTP response headers of the site, which revealed the server banner, status codes, cookies and redirect behavior. This is free technical intelligence about how the web application is built and served.

**5. wafw00f.** I detected the Web Application Firewall: the site sits behind **ModSecurity (SpiderLabs)**. Knowing a WAF is present shapes the whole attack plan, because naive attempts will be blocked or logged, so the attacker must adapt or attempt a bypass.

**6. dnsrecon.** I enumerated the full DNS footprint: name servers, mail servers, SPF policy, service (SRV) records, the DNS software version (**Bind 9.16.23**) and cPanel-related records. Each record is a potential foothold and reveals how email and hosting are set up.

## 4.2 Footprinting with the Google Hacking Database (W2-PM2)

GHDB (hosted on exploit-db.com) is a collection of ready-made search queries called **Google dorks**. These dorks combine normal Google operators in clever ways to surface sensitive information that organizations have accidentally left public, such as exposed camera feeds, open directories, login pages, config files and documents. It exists so that pentesters and defenders can find and fix these leaks.

**Task 1: Exposed security cameras.** Using dorks such as `intitle:"webcamXP" inurl:8080` and `intitle:"webcamxp 5" intext: "live stream"`, I located 10 live camera interfaces accessible from the internet:

| # | Link found | Dork used |
|---|---|---|
| 1 | http://109.233.191.130:8080/ | intitle:"webcamXP" inurl:8080 |
| 2 | http://72.199.200.5:8080/ | intitle:"webcamXP" inurl:8080 |
| 3 | http://139.64.168.120:8080/ | intitle:"webcamXP" inurl:8080 |
| 4 | http://109.233.191.130:8080/multi.html | intitle:"webcamXP" inurl:8080 |
| 5 | http://85.93.53.175:8080/gallery.html?page=6 | intitle:"webcamXP" inurl:8080 |
| 6 | http://www.seccam.mywire.org/ | intitle:"webcamXP" inurl:8080 |
| 7 | http://184.57.102.6:5432/ | intitle:"webcamxp 5" intext:"live stream" |
| 8 | https://www.skylinewebcams.com/webcam/italia/lazio/roma/piazza-venezia.html | inurl:webcam site:skylinewebcams.com inurl:roma |
| 9 | https://www.skylinewebcams.com/webcam/italia/lazio/roma/campo-de-fiori.html | inurl:webcam site:skylinewebcams.com inurl:roma |
| 10 | https://www.skylinewebcams.com/en/webcam/espana/canarias/santa-cruz-de-tenerife/playa-los-cristianos.html | inurl:webcam site:skylinewebcams.com inurl:roma |

**Task 2: Open directory listings.** Using the dork `intitle index of "parent directory" mathematics pdf`, I located 10 open directories exposing downloadable mathematics ebooks in PDF format:

| # | Link found | Dork used |
|---|---|---|
| 1 | https://lira.epac.to/DOCS-TECH/Math/Engineering%20and%20Applied/ | intitle index of "parent directory" mathematics pdf |
| 2 | https://www.fsr.ac.ma/DOC/cours/maths/ | intitle index of "parent directory" mathematics pdf |
| 3 | https://math.umd.edu/~immortal/206tutorial/ | intitle index of "parent directory" mathematics pdf |
| 4 | https://www.aetkin.com/files/Math%20150%20Calculus%20I/Advanced%20Calculus%20Textbook/ | intitle index of "parent directory" mathematics pdf |
| 5 | https://mathflix.luc.edu/pdfs/Algebra/ | intitle index of "parent directory" mathematics pdf |
| 6 | https://www.jsoftware.com/books/pdf/ | intitle index of "parent directory" mathematics pdf |
| 7 | https://netlib.org/math/docpdf/ | intitle index of "parent directory" mathematics pdf |
| 8 | http://erewhon.superkuh.com/library/Math/ | intitle index of "parent directory" mathematics pdf |
| 9 | https://math-olymp.rudn.ru/doc/ | intitle index of "parent directory" mathematics pdf |
| 10 | http://www.issp.ac.ru/ebooks/books/open/ | intitle index of "parent directory" mathematics pdf |

These links are recorded as lab evidence only. They demonstrate how a simple search query can reach systems that their owners believe are private.

## 4.3 Passive Enumeration with theHarvester (W2-PM4)

theHarvester gathers emails, sub-domains, hosts, employee names, open ports and banners from public sources such as search engines, PGP key servers and the SHODAN database, without ever touching the target directly.

I ran the tool against the lab organization domain microsoft.com in two passes:

1. **Baidu as the source, limit 1000:** `theHarvester -d microsoft.com -b baidu -l 1000`
2. **All sources, limit 50:** `theHarvester -d microsoft.com -b all -l 50`

Both passes returned email addresses, sub-domains and host names. Each harvested email is a ready-made phishing or password-spray candidate, and each sub-domain is another entrance that the defender may have forgotten exists. Because the tool only reads public data, the target never knows it is being studied, which is exactly why passive recon is so powerful and so hard to detect. Defenders should run theHarvester against their own organization to see what an attacker sees, and then shrink that footprint.

## 4.4 Network Scanning with Zenmap (W2-PM5)

For the scanning phase, I performed host discovery on my own local network:

1. **Local IP discovery.** On Windows, the `ipconfig` command identified my PC's local IP address and LAN subnet.
2. **Ping scan.** I entered my subnet into Zenmap and ran a **Ping Scan**, which found **2 live hosts** (including my own PC): `172.20.10.1` (the gateway) and `172.20.10.2` (my PC).
3. **MAC address collection.** The scan recorded the MAC address of the live hosts: `06:13:7A:64:D8:64`.
4. **Topology map.** I opened the Topology view, enabled the legend, and saved the network topology as a PDF.

From an attacker's perspective, this is the first move after landing inside a network: find the gateway and the neighbors, then decide who is worth attacking. From a defender's perspective, an unknown device on this list is the first sign of trouble.

# 5. Risk Analysis / Impact

Based on the information collected during these activities, I identified the following potential risks:

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Web technology and versions exposed | WhatWeb identified WordPress 7.0.4 and WP Download Manager 3.3.58 | Attackers can match exposed versions against vulnerability databases | Medium |
| 2 | Hosting provider identifiable | WHOIS name servers point to HostGator | Supports targeted social engineering and infrastructure attacks | Low |
| 3 | Server IP address resolvable | nslookup resolved the domain to 192.232.216.135 | Enables direct scanning of the hosting server | Low |
| 4 | HTTP technical detail exposed | curl -I returned response headers | Helps fingerprint the application stack | Low |
| 5 | WAF presence identified | wafw00f detected ModSecurity (SpiderLabs) | Reveals part of the security architecture, prompting bypass research | Low |
| 6 | Full DNS footprint mappable | dnsrecon enumerated NS, MX, SPF, SRV, Bind 9.16.23 and cPanel records | Each record is a potential foothold; email infrastructure exposed | Medium |
| 7 | Internet-accessible cameras discoverable | GHDB dorks surfaced 10 live camera interfaces | Unauthorized viewing of third-party feeds; default credentials are common | High* |
| 8 | Open directories exposing files | GHDB dorks surfaced 10 open directory listings | Sensitive documents downloadable by anyone who finds them | Medium* |
| 9 | Emails and sub-domains harvestable | theHarvester returned emails, sub-domains and hosts passively | Direct material for phishing and attacks on forgotten assets | Medium |
| 10 | Local hosts discoverable | Zenmap ping scan found 2 live hosts on my LAN | Network discovery is the first step of an internal attack | Low |

*Risks 7 and 8 concern third-party systems found through GHDB, recorded here only as educational evidence of how easily such exposure is found.

These are observations from reconnaissance and scanning exercises, not confirmed vulnerabilities. No exploitation or validation was performed. The presence of a software version, an IP address or a DNS record does not by itself mean that a system is vulnerable; further authorized testing would be required to confirm anything.

# 6. Recommendations

1. **Minimize publicly exposed technology details.** Organizations should regularly review what their websites reveal about servers, CMS platforms and plugins, and remove whatever is unnecessary.
2. **Keep CMS software and plugins updated** and check them against current security advisories, because exposed version numbers invite matching attacks.
3. **Harden HTTP response headers** so they do not hand attackers free technical intelligence.
4. **Review DNS records regularly** and expose only what is strictly required.
5. **Keep the WAF enabled and tuned.** Knowing ModSecurity is present deters naive attacks, and a well-tuned WAF stops more.
6. **Search for your own exposure with dorks.** Defenders should periodically run GHDB-style queries (cameras, directories, documents) against their own domains and fix whatever surfaces.
7. **Restrict directory listing** on web servers so that indexes of files are never shown.
8. **Run passive enumeration on yourself.** Tools like theHarvester show an organization its own leakable email addresses and sub-domains before an attacker collects them.
9. **Monitor for unknown devices.** Regular internal network discovery with tools like Zenmap and Nmap catches unauthorized devices early.
10. **Always test with authorization.** Reconnaissance and scanning belong only within a defined, permitted scope.

# 7. Conclusion

Week 2 covered the two phases that decide most real intrusions before they happen: reconnaissance and network discovery.

I learned to footprint a domain with six different Kali tools, with each tool answering a different question: who registered it, what software it runs, where it lives, how it answers requests, what protects it, and how its DNS is structured. I learned that Google itself is a hacking tool, because a few well-chosen operators exposed live cameras and open directories that their owners would insist are private. I practiced passive enumeration with theHarvester, and I scanned my own network with Zenmap to understand how quickly an attacker inside a LAN can map who is present.

My biggest takeaway: most of an attacker's advantage is built from information that nobody bothered to hide. Reconnaissance is quiet, cheap and hard to detect, and defenders can only respond by seeing their own exposure the way an attacker would, then shrinking it.

Finally, I learned that findings only matter when they are communicated properly: what was performed, what was discovered, what it means, what risk it creates, and what should be fixed. That is what this report practices.

# 8. Evidence Collected

Screenshots collected during the activities are stored in the `screenshots/` folder:

| File | Evidence |
|---|---|
| screenshots/01_whois.png | WHOIS domain registration output |
| screenshots/02_whatweb.png | WhatWeb technology fingerprint |
| screenshots/03_nslookup.png | nslookup DNS resolution |
| screenshots/04_curl.png | curl -I HTTP response headers |
| screenshots/05_wafw00f.png | wafw00f WAF detection |
| screenshots/06_dnsrecon.png | dnsrecon DNS record enumeration |
| screenshots/07_theharvester_baidu.png | theHarvester, Baidu source, limit 1000 |
| screenshots/08_theharvester_all_sources.png | theHarvester, all sources, limit 50 |
| screenshots/09_ipconfig.png | Local IP address and subnet identification |
| screenshots/10_zenmap_ping_scan.png | Zenmap ping scan results, live hosts and MAC addresses |
| screenshots/11_topology.pdf | Zenmap network topology export |

The GHDB findings in section 4.2 serve as their own evidence, in the dork tables above.

-End-
