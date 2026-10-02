# Building an Isolated Lab Network for Metasploitable

**Platform:** VMware Workstation Pro, Kali Linux, Metasploitable 2 | **Category:** Networking / Lab Setup | **Difficulty:** Beginner | **Date:** 02/10/2026

## Objective
Build a private, isolated network between Kali and a deliberately
vulnerable target (Metasploitable), so it can be attacked safely
without being reachable from, or reaching, anything else.

## Approach
I set up Kali with two network adapters — NAT for internet access,
and Host-only for a private link to Metasploitable. When I checked
Metasploitable's network settings, I found its primary adapter was
set to **Bridged**, not Host-only. Bridged mode puts a VM directly on
the real physical network, so Metasploitable had picked up an address
from my actual router (`192.168.18.138`) rather than the isolated lab
network — meaning a deliberately unpatched, vulnerable machine was
reachable by anything else on my home network.

Separately, I noticed Kali's own NAT adapter (`eth0`) had no IP
address at all. Checking with `nmcli device status` showed it had no
connection profile assigned, so it was sitting disconnected. I created
one manually and brought it up.

## Key commands
```bash
# Check which interfaces are managed and their state
nmcli device status

# Create a missing connection profile for the NAT adapter
sudo nmcli connection add type ethernet ifname eth0 con-name eth0-nat
sudo nmcli connection up eth0-nat

# Confirm addressing on both interfaces
ip a

# Verify internet still works via NAT
ping -c 4 8.8.8.8

# Verify the isolated link to Metasploitable works
ping -c 4 192.168.5.5
```

## Results
| Interface | Network | Address | Purpose |
|---|---|---|---|
| Kali eth0 | NAT (VMnet8) | 192.168.153.128 | Internet access |
| Kali eth1 | Host-only (VMnet1) | 192.168.5.4 | Lab link to target |
| Metasploitable eth0 | Host-only (VMnet1) | 192.168.5.5 | Isolated target |

## What was actually vulnerable
Metasploitable is deliberately full of old, unpatched services. On
its own that's fine inside an isolated lab — that's the whole point of
the machine. The real problem was the adapter setting: with it bridged
onto my real network, any other device on that network could
potentially have reached it, turning a safe training target into a
genuine exposure.

## How it would be fixed
Set the adapter to Host-only before ever powering the VM on, and
verify the resulting IP address is actually in the expected private
range — not just assume a setting is correct because it looks right
in the VM's settings panel.

## What I learned
- A network "looking" isolated in the settings menu isn't the same as
  verifying it actually is — checking the real IP address with
  `ifconfig`/`ip a` is the only way to be sure.
- A missing NetworkManager connection profile is a common, silent
  cause of an interface showing no IP — `nmcli device status` reveals
  it immediately.
- Lab setup mistakes are security mistakes. Catching a misconfigured
  adapter before attacking the target is the same skill as catching a
  misconfiguration on a real engagement.
