# Task 3 — Vulnerability Scan & Password Brute-Force

## 1. Objective

Scan the tc-corp-server host for exposed services, create a password wordlist, and use hydra to brute-force one of the exposed services. Document the scan, the wordlist creation, and the brute-force outcome.

**Target:** tc-corp-server lab VM (IP: 10.10.1.246), from Kali attacker workstation (IP: 10.10.1.6), isolated network.

## 2. Methodology (step by step)

1. Ran a full TCP port/service-version scan against the target: `nmap -sV -p- 10.10.1.246`. This enumerated every one of the 65,535 TCP ports and attempted to fingerprint the service and version behind each open one.
2. The scan returned a very large attack surface — over 30 open ports/services, including `ftp` (21), `ssh` (22, OpenSSH 10.0p2), `telnet` (23), `smtp` (25, 587), `http`/`https` (80, 443, 8080, 8081, 9000), `pop3`/`pop3s` (110, 995), `rpcbind` (111), `netbios-ssn`/Samba (139, 445), `imap`/`imaps` (143, 993), `ldap` (389), `ipp`/CUPS (631), `rsync` (873), `mosquitto` MQTT (1883), `nfs` (2049), a second `ftp` (2121), an `http-proxy` (Squid, 3128), `mysql`/MariaDB (3306), `distccd` (3632), `postgresql` (5432), `redis` (6379), `irc` (ngircd, 6667), `memcached` (11211), plus several RPC/mount-related services.
3. Chose **ssh (port 22, OpenSSH 10.0p2)** as the service to brute-force. Justification: SSH is a realistic, commonly-targeted remote-administration service with credential-based authentication (unlike, say, a read-only service), and a successful compromise here would represent genuine remote shell access to the host — a higher-impact outcome than most of the other exposed services.
4. First attempt at wordlist creation: generated a purely numeric candidate list with `crunch 4 6 0123456789 -o wordlist.txt`, producing 1,110,000 four-to-six-digit numeric candidates (~7.65 MB). The reasoning was to cover PIN-style numeric passwords quickly.
5. Ran `hydra -l root -P wordlist.txt ssh://10.10.1.246` against this list. In practice this proved impractical within the exam timeframe: SSH's per-attempt connection/authentication overhead means testing 1.1 million candidates would take an unreasonably long time, so the run was stopped before completion rather than left to exhaust the list.
6. Pivoted to a smaller, hand-crafted wordlist instead, built directly in `nano` (`small-wordlist.txt`): 11 common/weak passwords often seen on poorly-secured or default lab systems — `password`, `admin`, `root`, `toor`, `12345`, `12345678`, `kali`, `password123`, `letmein`, `qwerty`, `corp`. This targeted plausible weak credentials rather than exhaustively covering a large keyspace.
7. Ran `hydra -l root -P small-wordlist.txt -t 4 -w 3 ssh://10.10.1.246` (4 parallel tasks, 3-second wait between retries, to stay within sensible limits).
8. The run completed cleanly: **1 of 1 target completed, 0 valid passwords found.** No password from the wordlist matched the `root` account over SSH.

## 3. Payloads / commands tried (in order)

| # | Command | Purpose | Result |
|---|---|---|---|
| 1 | `nmap -sV -p- 10.10.1.246` | Enumerate open ports/services | 30+ open ports/services identified, including ssh, ftp, telnet, http(s), smtp, databases, and more |
| 2 | `crunch 4 6 0123456789 -o wordlist.txt` | Generate a numeric wordlist | Generated 1,110,000 candidates (4–6 digit numeric strings), ~7.65 MB |
| 3 | `hydra -l root -P wordlist.txt ssh://10.10.1.246` | Brute-force ssh with the numeric wordlist | Impractically slow against 1.1M entries given SSH's per-attempt latency; stopped before completion |
| 4 | `nano small-wordlist.txt` | Hand-craft a small, targeted wordlist of common weak passwords | 11-entry wordlist created |
| 5 | `hydra -l root -P small-wordlist.txt -t 4 -w 3 ssh://10.10.1.246` | Brute-force ssh with the targeted wordlist | Completed: **0 valid passwords found** from this list |

## 4. Evidence (screenshots)

![01 — nmap scan results](screenshots/01-nmap-scan.png)
Note: Full TCP scan of 10.10.1.246 showing 30+ open ports and fingerprinted service versions, including ssh (OpenSSH 10.0p2).

![02 — crunch wordlist generation](screenshots/02-crunch-wordlist.png)
Note: `crunch 4 6 0123456789 -o wordlist.txt` generating a 1,110,000-line numeric candidate list.

![03 — initial hydra run against the large wordlist](screenshots/03-hydra-large-wordlist-attempt.png)
Note: hydra launched against the 1.1M-entry numeric wordlist; stopped as impractically slow for the exam timeframe.

![04 — hand-crafted wordlist in nano](screenshots/04-nano-small-wordlist.png)
Note: 11-entry targeted wordlist of common weak passwords, built directly in nano.

![05 — hydra result against the targeted wordlist](screenshots/05-hydra-small-wordlist-result.png)
Note: hydra completed its run against small-wordlist.txt — "1 of 1 target completed, 0 valid password found."

## 5. Analysis

The nmap scan revealed a strikingly large attack surface for a single host — over 30 services exposed simultaneously, spanning web, mail, file-sharing, database, and remote-administration protocols. On an intentionally vulnerable training host this is expected, but on any real system it would itself be a significant finding: every open, unnecessary service is additional attack surface regardless of whether any individual one is successfully exploited.

SSH was a reasonable brute-force target precisely because it offers direct administrative access if credentials are guessed correctly. The first wordlist-generation approach (an exhaustive numeric range via `crunch`) illustrates a real trade-off in brute-forcing: broad, unguided coverage produces an enormous keyspace that is often impractical to test against a network service with per-attempt latency, whereas a smaller, intelligence-led list (common/default/weak passwords) tests far fewer candidates but is more likely to hit a real-world weak credential quickly. In this case, the targeted 11-password list did not succeed against the `root` account — a negative result, but still a valid, well-reasoned documented attempt per the exam's own guidance, and it demonstrates that this particular account is not using one of the most common weak passwords. It does not, however, rule out a different weak password outside the tested list, nor confirm whether account lockout or rate-limiting defences are in place on this host — testing that would be a logical next step.

## 6. Remediation / defence

- **Disable direct root login over SSH** (`PermitRootLogin no` in `sshd_config`), forcing privilege escalation through a named, individually-accountable account instead.
- **Move to key-based authentication** instead of passwords for SSH wherever possible, which makes online password brute-forcing irrelevant for that service.
- **Rate-limit and lock out repeated failed attempts** (e.g. `fail2ban`, or SSH's own `MaxAuthTries`/connection throttling) so automated brute-force tools like hydra are slowed to the point of impracticality.
- **Enforce a strong password policy and/or MFA** for any account that must retain password authentication.
- **Reduce the exposed attack surface found during the scan** — close or firewall off services not genuinely needed (e.g. telnet, rpcbind, exposed databases like MySQL/PostgreSQL/Redis, memcached) rather than leaving them reachable by default; this is a broader, independent finding from the scan itself, separate from the brute-force result.
