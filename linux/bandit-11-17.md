# OverTheWire Bandit — Levels 11 to 17

**Platform:** OverTheWire | **Category:** Linux / CLI | **Difficulty:** Beginner | **Date:** 03/10/2026

## Objective
Continue through Bandit, moving from file handling into encoding layers,
key-based authentication, and network tools.

## Level summary

| Level | Concept | Key command |
|-------|---------|-------------|
| 11 → 12 | Rotating every letter by 13 | `tr 'A-Za-z' 'N-ZA-Mn-za-m'` |
| 12 → 13 | Undoing a hex dump, then repeated compression layers | `xxd -r`, then `file` / `mv` / decompress, repeated |
| 13 → 14 | Logging in with an SSH private key | `ssh -i <private_key>` |
| 14 → 15 | Sending data to a port with netcat | `nc localhost <port>` |
| 15 → 16 | Talking to a port over SSL/TLS | `openssl s_client -connect localhost:<port>` |
| 16 → 17 | Scanning a port range for the real service | `nmap -p <port-range> -sV localhost` |
| 17 → 18 | Comparing two similar files | `diff passwords.old passwords.new` |

## What I learned
**ROT13:**
ROT13 replaces each letter with the one 13 places along the alphabet. 
It's technically a Caesar cipher, but with a fixed, 
public shift and no secret key, anyone can reverse it instantly. 
It hides text from a casual glance, not from anyone who looks.
**SSH private-key authentication:**
The private key is never sent to the server. 
Instead, the SSH client proves that it possesses the private key by signing data during authentication, 
and the server verifies that signature using the corresponding public key. 
I also learned that SSH enforces strict permissions on private keys, 
so a key may be rejected if it is too accessible; chmod 600 fixes the permissions.
**SSL/TLS vs. plain netcat:**
nc can connect to a TCP port and send raw data, but it cannot perform the SSL/TLS handshake required
by an encrypted service. openssl s_client handles that handshake and lets you communicate with the service securely.
**Why port scanning was necessary:**
In Level 16, the correct service was hidden somewhere in a range of ports, so guessing wasn't practical.
Scanning the range with Nmap let me identify which ports were open and
then determine which one was actually providing the expected service.

## Where I got stuck
Level 12 → 13 was the level where I got stuck the most. I initially understood that the file had been hex-dumped,	
but the confusing part was realizing that restoring the hex dump was only the first step. 
After using xxd -r, the resulting file could still be compressed, and 
I had to repeatedly identify the file type, rename it with the appropriate extension,
and decompress it layer by layer. It clicked when I understood that I wasn't looking for one command 
to solve the whole file — I was following a repeated cycle of identify → rename → decompress → identify again.

Level 16 → 17: I initially found several open ports in the given range and 
assumed that any port accepting an SSL connection might be the answer. 
After connecting to them with openssl s_client, 
I found that several ports did speak SSL, 
but they did not all provide the expected service or useful authentication material. 
The important part was learning to distinguish between an open SSL service and 
the specific service the level was asking me to find. 
Once I inspected the responses from the different ports, 
I could identify the one that actually provided something useful for the next level.
---
No passwords are included in this writeup.
