# Scanning Metasploitable — A Lesson in Verifying the Path

**Platform:** Metasploitable 2 (isolated lab) | **Category:** Networking | **Difficulty:** Beginner | **Date:** 03/10/2026

## Objective
Scan a deliberately vulnerable target inside my own isolated lab, compare it
with the earlier scanme.nmap.org scan, and identify which services stand out
as security concerns.

## Approach
I scanned Metasploitable (192.168.5.5) and got 23 open ports with real
version strings, unlike scanme, which returned `tcpwrapped`. Since
Metasploitable has no firewall, `-sV` worked cleanly.

A few hours later I re-ran the same scan and got a different result: the
977 closed ports had become filtered, and the ping TTL had changed from 64
to 128. The services themselves were unchanged, so I investigated the path
rather than the target.

Using `--reason`, `ip neigh`, and `ip route get`, I found that my host-only
adapter (`eth1`) had lost its connection profile and come up disconnected.
Traffic to 192.168.5.5 was going out through the NAT adapter (`eth0`) to
VMware's NAT gateway instead of directly over the host-only link. The TTL
128 and the filtered ports both came from that detour, not from anything on
the target.

I recreated and pinned a connection profile for `eth1`, confirmed the route
went directly to the target again, and re-ran the scan. It matched the
original result exactly: same 23 ports, same versions, TTL back to 64.

**Note on the detour:** while `eth1` was down, scans still reached the
target via the Windows host's own adapter on the host-only network — this
is expected host-only behaviour, not a lab fault. I confirmed real isolation
separately by checking that Metasploitable has no route to the internet.

## Key commands
```bash
# Initial version scan
nmap -sV 192.168.5.5

# Investigate why closed ports became filtered
sudo nmap -sS --reason -p 20-30,1000-1010 192.168.5.5

# Check whether Kali has a direct ARP entry for the target
ip neigh show 192.168.5.5

# Check which interface traffic to the target actually uses
ip route get 192.168.5.5

# Recreate and pin the missing connection profile
sudo nmcli connection add type ethernet ifname eth1 con-name eth1-hostonly
sudo nmcli connection modify eth1-hostonly connection.interface-name eth1
sudo nmcli connection up eth1-hostonly

# Confirm the fix
ping -c 2 192.168.5.5
nmap -sV 192.168.5.5
```

## Results

| State | Route | Ping TTL | Port 1–1000 (not shown) |
|---|---|---|---|
| Before (working) | `dev eth1`, direct | 64 | 977 closed (reset) |
| During fault | `via 192.168.153.2 dev eth0` | 128 | 977 filtered (no-response) |
| After fix | `dev eth1`, direct | 64 | 977 closed (reset) |

Open ports and versions were identical in all three scans, confirming it
was the path that changed, not the target.

## What was actually vulnerable

Metasploitable 2 is deliberately insecure, so the point here is reading the
scan and knowing why each service stands out — not exploiting anything.
Each row below is a hypothesis from the `-sV` output, checked against
`searchsploit` where possible.

| Port | Service (from `-sV`) | Why it stands out | How it would be fixed |
|------|----------------------|--------------------|-------------------------|
| 1524 | bindshell | Nmap identifies this as a root shell. A shell listening on a network port with no authentication. | Remove the service and block the port. |
| 21 | vsftpd 2.3.4 | Confirmed via `searchsploit`: this exact version was distributed with a backdoor (CVE-2011-2523). Exploit modules exist for both Metasploit and standalone Python. | Upgrade to a patched release. Prefer SFTP. |
| 6667 | UnrealIRCd | `searchsploit` lists the vulnerable build as 3.2.8.1. Nmap didn't print a version, so this is a hypothesis to confirm via banner grab, not a confirmed match. | Confirm the build, then upgrade or remove if unused. |
| 23, 512, 513, 514 | telnet, rexec, rlogin, rsh | Cleartext remote access. The r-services rely on host-based trust rather than real authentication. | Disable them and use SSH instead. |
| 3306, 5432 | MySQL 5.0.51a, PostgreSQL 8.3.x | Databases reachable over the network, on very old versions. | Bind to localhost, firewall the ports, upgrade. |
| 5900 | VNC (protocol 3.3) | Remote desktop exposed with an old, weak protocol version. | Tunnel through SSH or remove it. |
| 139, 445 | Samba 3.x | Old SMB. Nmap only gave a version range, so no specific CVE can be assigned yet. | Identify the exact version, then upgrade. |
| 22 | OpenSSH 4.7p1 | Roughly 2007-era software, long out of support. | Upgrade the OS and SSH. Use key-only authentication. |

Overall the target offers 23 open services on an out-of-support OS, several
in cleartext. The broader lesson is attack surface: every unnecessary
listening service is one more thing to patch, monitor, and defend.

## What I learned
- **A scan result reflects the path as well as the target.** The same
  machine read as closed-then-reset and later as filtered-then-silent,
  purely because of a routing change on my own machine.
- **TTL is a quiet but reliable clue.** A jump from 64 (Linux default) to
  128 (Windows default) was the first sign that traffic was being handled
  by something other than the target.
- **An empty `ip neigh` entry means you're not talking to the host
  directly** — on a local subnet, a direct connection always leaves an ARP
  entry behind.
- **Confirm a finding with a second tool before trusting it.** `searchsploit`
  verified the vsftpd backdoor and also corrected my assumption about which
  UnrealIRCd build was vulnerable.

---
No credentials or exploit payloads were used. Nothing beyond scanning and
version/banner identification was performed against this target.
