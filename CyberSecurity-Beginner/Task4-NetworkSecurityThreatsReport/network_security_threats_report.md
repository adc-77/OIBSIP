# Common Network Security Threats: How They Work, Real-World Impact, and Mitigation

**Author:** Ayush kumar Dubey 
**Date:** July 2026

---

## Introduction

Every device that connects to a network — a laptop, a phone, a smart camera, a server sitting in a data center — is a potential entry point for someone who wants to disrupt, spy on, or hijack that network. As organisations move more of their operations online and rely on interconnected systems for everything from payments to healthcare records, the cost of a successful network attack has grown from "annoying outage" to "multi-million-dollar incident" in a very short span of time. Understanding how these attacks actually work, not just their names, is what separates a network administrator who can respond to an incident from one who is simply waiting for it to happen. This report walks through four of the most common network security threats — DoS/DDoS attacks, Man-in-the-Middle attacks, IP spoofing, and DNS poisoning — using real incidents to show what's at stake and what specifically can be done to reduce the risk.

---

## 1. DoS/DDoS Attacks

**How it works:**
A Denial-of-Service (DoS) attack tries to make a service unavailable to its legitimate users, usually by overwhelming it with more traffic or requests than it can handle. A single machine can do this against a small target, but most modern attacks are Distributed Denial-of-Service (DDoS) attacks, where traffic comes from thousands or even millions of compromised devices at once (a "botnet"), making the flood far larger and much harder to block, since the traffic appears to come from everywhere rather than one identifiable source. Attackers also use amplification techniques, where a small spoofed request to a misconfigured server (like an open DNS resolver or a memcached server) tricks that server into sending a much larger reply to the victim, multiplying the attacker's effort by dozens or even hundreds of times.

**Real-world example:**
In October 2016, the Mirai botnet was used to attack Dyn, a company that provided DNS services for major platforms including Twitter, Netflix, Reddit, PayPal, and Amazon. Mirai had spread by scanning the internet for IoT devices — routers, DVRs, IP cameras — still using their factory default passwords, and turning them into remote-controlled bots. When the botnet was pointed at Dyn, it generated a flood of malicious DNS lookup requests from tens of millions of IP addresses. Because so many major websites depended on Dyn to resolve their domain names, the attack effectively knocked large parts of the internet offline across the US and Europe for several hours, even though none of those companies' own servers were directly attacked.

**Impact:**
Beyond the obvious service outage, DDoS attacks like this one cause real financial damage — lost revenue during downtime, the cost of emergency mitigation, and reputational harm when customers can't reach a service they depend on. The Dyn incident also served as a wake-up call about how insecure consumer IoT devices could be weaponised at internet scale.

**Mitigation strategies:**
1. Deploy a DDoS protection/scrubbing service (e.g., a CDN or dedicated anti-DDoS provider) that can absorb and filter volumetric traffic before it reaches origin servers.
2. Use rate limiting and traffic shaping at the network edge to cap the number of requests any single source can make in a given time window.
3. Harden IoT and network devices by changing default credentials and disabling unnecessary remote-access services, reducing the pool of devices attackers can recruit into botnets in the first place.

---

## 2. Man-in-the-Middle (MITM) Attacks

**How it works:**
In a MITM attack, the attacker secretly positions themselves between two parties who believe they are communicating directly with each other — intercepting, and sometimes altering, the traffic passing between them. This can happen at several levels: on a local network through ARP spoofing (tricking devices into sending traffic through the attacker's machine), through a fake or compromised Wi-Fi access point, or at a much larger scale by compromising the certificate infrastructure that browsers rely on to trust a website's identity.

**Real-world example:**
In 2011, DigiNotar, a Dutch certificate authority, was breached by an attacker who used the company's own systems to issue several hundred fraudulent SSL certificates for domains including Google, Mozilla, and Skype. Because browsers trusted DigiNotar as a legitimate certificate authority, those fraudulent certificates allowed whoever held them to impersonate Google and other sites convincingly, with no warning shown to the user. The certificates were later used to intercept the Gmail traffic of a large number of users in Iran. Once the fraud was discovered, every major browser revoked trust in DigiNotar's certificates, and the company was forced into bankruptcy within months.

**Impact:**
This incident showed that even encrypted (HTTPS) traffic is only as trustworthy as the certificate authority that vouches for it — a single compromised CA can undermine the security assumptions of the entire web for its users. Beyond this case, MITM attacks are routinely used to steal login credentials, session cookies, and financial information on unsecured public Wi-Fi networks.

**Mitigation strategies:**
1. Enforce HTTPS everywhere and use HSTS (HTTP Strict Transport Security) so browsers refuse to fall back to an unencrypted connection.
2. Use certificate pinning for sensitive applications, so the app only trusts a specific, known certificate rather than any certificate a compromised CA might issue.
3. Avoid transmitting sensitive data over public or unknown Wi-Fi networks without a trusted VPN, and use network monitoring tools to detect ARP spoofing on internal networks.

---

## 3. IP Spoofing

**How it works:**
IP spoofing means forging the source IP address in a packet's header so that it appears to come from a different, often trusted, machine. Because the core IP protocol was never designed with strong authentication in mind, a machine receiving a packet has no built-in way to verify that the "from" address is genuine. Attackers use this to disguise the true origin of malicious traffic, to bypass IP-based access controls, or to redirect the response to a packet toward a third-party victim instead of themselves — which is exactly how amplification-based DDoS attacks work.

**Real-world example:**
One of the earliest and most famous cases was the 1994 attack by Kevin Mitnick against security researcher Tsutomu Shimomura. Mitnick spoofed the IP address of a machine that Shimomura's target server already trusted, predicted the TCP sequence numbers the server would use (a weakness in how TCP connections worked at the time), and used this blind spoofing technique to establish a fraudulent trusted connection and plant a backdoor — all without ever seeing the server's actual responses. More recently, IP spoofing was central to the February 2018 attack on GitHub, where attackers spoofed GitHub's IP address in requests sent to publicly exposed memcached servers; each server amplified the reply by roughly 50 times, generating a peak of 1.35 terabits per second of traffic directed at GitHub, which was one of the largest DDoS attacks recorded at that time.

**Impact:**
IP spoofing rarely causes damage on its own — its danger comes from what it enables: undetectable DDoS amplification, bypassing firewalls that trust specific IP ranges, and hiding the attacker's true identity from investigators.

**Mitigation strategies:**
1. Implement ingress and egress filtering (BCP 38) at network borders so routers drop packets whose source address couldn't legitimately have originated from that network segment.
2. Use modern, properly randomised TCP sequence numbers and avoid trust relationships based solely on IP address (the exact weakness Mitnick exploited).
3. Disable or firewall off unauthenticated UDP services like open memcached or DNS resolvers that can be abused for spoofed amplification attacks.

---

## 4. DNS Poisoning / Spoofing (Bonus Threat)

**How it works:**
DNS poisoning (also called DNS cache poisoning) involves inserting fraudulent DNS records into a resolver's cache, so that when a user looks up a legitimate domain name, they are instead handed the IP address of a server controlled by the attacker. Because DNS traditionally runs over UDP with only a 16-bit transaction ID for verification, an attacker who can guess or brute-force that ID before the real response arrives can trick a resolver into caching a fake answer — an approach that was dramatically demonstrated by security researcher Dan Kaminsky in 2008, when he showed that querying for random non-existent subdomains could be used to poison an entire domain's resolution far more easily than previously thought.

**Real-world example:**
In April 2018, attackers used a BGP hijack to reroute traffic destined for Amazon's Route 53 DNS service, targeting the authoritative name servers for the cryptocurrency wallet site MyEtherWallet.com. For roughly two hours, users looking up MyEtherWallet were sent to a phishing server that closely mimicked the real site, and even though a certificate warning appeared, many users clicked through it anyway. The attackers made off with an estimated $17 million in stolen Ethereum.

**Impact:**
DNS poisoning is especially dangerous because it's largely invisible to the end user — the browser's address bar still shows the correct domain name, so victims have no obvious reason to be suspicious. It's been used for credential theft, malware distribution, and, in some documented cases, government-directed censorship or surveillance by manipulating DNS responses at the ISP level.

**Mitigation strategies:**
1. Deploy DNSSEC, which cryptographically signs DNS records so resolvers can verify a response actually came from the legitimate authoritative server.
2. Use source port randomisation and query name randomisation (0x20 encoding) on DNS resolvers to make blind cache-poisoning attempts far harder to pull off.
3. Monitor BGP routing announcements for your critical infrastructure and use RPKI (Resource Public Key Infrastructure) to detect and reject unauthorised route hijacks before they can be used to redirect DNS traffic.

---

## Comparison Table

| Threat | Primary Attack Vector | Who Is at Risk | Difficulty to Execute | Ease of Mitigation |
|---|---|---|---|---|
| DoS/DDoS | Traffic flooding, often via botnets or amplification | Any internet-facing service; especially high-traffic or single-point-of-failure infrastructure like DNS providers | Low–Medium (botnets and amplification tools are widely available) | Medium (requires dedicated scrubbing/CDN capacity, not just local config) |
| Man-in-the-Middle | Interception via ARP spoofing, rogue Wi-Fi, or compromised certificates | Users on public/unsecured networks; any organisation relying on a single trusted CA | Medium (local network attacks are easy; CA-level compromise is hard) | Medium (HTTPS/HSTS help a lot, but public Wi-Fi habits are hard to control) |
| IP Spoofing | Forging the source IP address in packet headers | Networks with weak border filtering; services trusting IP-based access rules | Medium (blind spoofing needs some skill; enables larger attacks) | High (BCP 38 filtering is a well-established, effective fix) |
| DNS Poisoning/Spoofing | Injecting fraudulent records into DNS caches or hijacking routes | Any organisation without DNSSEC; users of any service relying on that DNS | Medium–High (modern resolvers are harder to poison, but route hijacking still works) | Medium (DNSSEC adoption is still incomplete industry-wide) |

---

## Conclusion

Three takeaways stand out for anyone responsible for defending a network:

1. **Trust is the thing attackers exploit, not just technology.** Whether it's a certificate authority, a "trusted" IP address, or a DNS resolver's cache, every one of these attacks succeeds by abusing an assumption that something can be trusted without verification. Building in cryptographic verification (DNSSEC, certificate pinning, HTTPS everywhere) closes that gap.
2. **The weakest device on your network can compromise the strongest one.** The Dyn attack didn't succeed because Dyn's own servers were poorly defended — it succeeded because millions of unrelated, poorly secured consumer devices could be recruited against it. Basic hygiene (changing default credentials, patching, network segmentation) matters even for devices that don't seem important.
3. **Defense needs to happen at multiple layers.** No single control stops all four of these threats. A resilient network combines border filtering (BCP 38), encrypted and verified communication (HTTPS/HSTS/DNSSEC), monitoring (for ARP anomalies, BGP hijacks, and unusual traffic spikes), and a plan for absorbing an attack that does get through.

---

## References

1. National Institute of Standards and Technology (NIST). *Guide to DDoS Attacks* and related Special Publications on network security. https://www.nist.gov
2. Cybersecurity and Infrastructure Security Agency (CISA). Advisories on DNS security, BGP hijacking, and DDoS mitigation guidance. https://www.cisa.gov
3. SANS Institute Reading Room. Whitepapers on man-in-the-middle attacks, IP spoofing, and DNS cache poisoning. https://www.sans.org/reading-room
4. MITRE ATT&CK Framework. Techniques for Network Denial of Service (T1498), Adversary-in-the-Middle (T1557), and DNS-related techniques. https://attack.mitre.org
5. Wikipedia. "DDoS attacks on Dyn" and "Man-in-the-middle attack" — used for incident timeline verification. https://en.wikipedia.org
