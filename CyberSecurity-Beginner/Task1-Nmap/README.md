# Task 1 – Basic Network Scanning with Nmap

## Objective

Perform basic network scanning using Nmap to identify open ports, running services, service versions, and the operating system of an authorized target.

---

## Tools Used

- Ubuntu Linux
- Nmap

---

## Target

scanme.nmap.org (Official Nmap testing host)

---

## Commands Used

```bash
nmap --version
```

```bash
nmap scanme.nmap.org
```

```bash
sudo nmap -sV scanme.nmap.org
```

```bash
sudo nmap -O scanme.nmap.org
```

---

## What is Nmap?

Nmap (Network Mapper) is an open-source network scanning tool used to discover hosts, detect open ports, identify running services, determine service versions, and perform operating system detection.

---

## Why Network Scanning Matters

Network scanning helps administrators:

- Discover active systems
- Detect exposed services
- Identify unnecessary open ports
- Improve overall network security

---

## Ethical Guidelines

Only scan systems that you own or systems where you have explicit permission to perform testing. This task used the official Nmap testing host intended for learning and testing.

---

## Files Included

- README.md
- nmap_scan_results.txt
- screenshots/
