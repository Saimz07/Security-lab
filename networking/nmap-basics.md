# Nmap Basics — Scanning an Authorised Target

**Platform:** scanme.nmap.org | **Category:** Networking | **Difficulty:** Beginner | **Date:** 25/09/2026

## Objective
Scan a host I'm permitted to scan, and learn to interpret the results
rather than just run the tool.

## Approach
I first ran the default command `nmap -sV scanme.nmap.org`, but it took too
long because packets were being dropped and Nmap slowed itself down, so I
switched to the `-F` flag, which scans fewer ports.

That scan finished, but `-sV` reported both ports as open and `tcpwrapped`,
which means the connection was closed before Nmap could identify the version.
I suspected something between the target and me was interfering, so I checked
manually using `nc` and `curl`.

Both returned full version banners, so the services were running normally —
it was Nmap's probes that were being disrupted.

## Key commands
```bash
# Full version scan of the top 1000 ports — stalled due to dropped packets
nmap -sV scanme.nmap.org

# Top 100 ports only, to finish faster
nmap -sV -F scanme.nmap.org

# Manual banner grab on SSH, to verify what Nmap couldn't read
nc -v scanme.nmap.org 22

# Fetch only the HTTP response headers, to read the Server banner
curl -I http://scanme.nmap.org
```

## Results
| Port | State | Nmap -sV said | Manual check found |
|------|-------|---------------|--------------------|
| 22/tcp | open | tcpwrapped | OpenSSH 6.6.1p1 (Ubuntu) |
| 80/tcp | open | tcpwrapped | Apache 2.4.7 (Ubuntu) |

98 other ports: filtered.

## What was actually vulnerable
Both banners point to Ubuntu 14.04, which is very old. Old software has
well-known, publicly documented vulnerabilities, which is why regular updates
and patches matter.

When a server reveals its software and version, it hands an attacker
information that helps them identify possible weaknesses.

## How it would be fixed
- Review the SSH configuration and restrict access to authorised users and
  trusted networks where possible.
- Update the OS to a supported version, since old versions are vulnerable.
- Suppress version banners so the server stops announcing what software it runs.

## What I learned
- **Closed vs filtered:** closed means the target is reachable but nothing is
  listening on that port — it replies with a RST. Filtered means a firewall or
  filter is dropping the probes, so nothing comes back and Nmap can't tell
  whether the port is open or closed.
- **SYN scans (`-sS`)** require root privileges because they send and analyse
  raw TCP packets, which Linux restricts to privileged users.
- **One scan is often not enough.** The manual checks revealed the SSH and
  Apache versions where Nmap only reported `tcpwrapped`, which made it possible
  to identify the software and assess potential vulnerabilities.
