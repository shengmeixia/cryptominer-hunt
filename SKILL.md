---
name: cryptominer-hunt
description: Detect, investigate, and remove cryptocurrency miners and associated malware from a Linux server. Use when a server has high CPU usage, suspected mining activity, or after a compromise.
argument-hint: [--dry-run]
allowed-tools: Bash, Read, Glob, Grep, Agent
effort: max
---

# Cryptominer Hunt

You are performing incident response on a Linux server suspected of running cryptocurrency miners or other malware. Your mission: find everything malicious, understand how it got there, remove it, and ensure it doesn't come back.

**Mode:** If `$ARGUMENTS` contains `--dry-run`, report only — do not kill, delete, or modify anything. Otherwise, confirm with the user before destructive actions.

## How to work

- **Adapt to the environment.** Start by understanding what OS, architecture, tools, and services are present. Every server is different — choose your commands accordingly.
- **Follow the evidence.** Each finding should lead to deeper investigation. A miner implies a dropper; a dropper implies an entry point; an entry point implies persistence. Chase the full chain.
- **Work in parallel.** Run independent checks simultaneously. Report brief summaries between phases.
- **Think adversarially.** Attackers hide things. Check where you wouldn't normally look. Consider what you might be missing.

---

## Phase 1: Triage

**Goal:** Quickly determine if the server is compromised and identify the most obvious indicators.

Investigate these areas — the specific commands are up to you, but here's what to look for:

- **CPU anomalies** — What's consuming resources? Miners typically peg multiple cores.
- **Suspicious processes** — Look for unknown binaries, processes masquerading as kernel threads (brackets but wrong PPID), processes running from deleted files, processes with suspicious command-line arguments (pool URLs, wallet addresses, `--donate`, `--algo`).
- **Unusual network activity** — Outbound connections to unknown IPs, connections on common mining ports (3333, 4444, 5555, 14444, 45700, etc.), TLS connections to non-standard ports.
- **Files that shouldn't exist** — Executables in /tmp, /dev/shm, /var/tmp, hidden files in writable directories, recently created binaries in unusual locations.
- **Container-level activity** — If Docker/Podman/CRI is present, check for high-CPU containers, suspicious images, or containers running with host namespaces.

---

## Phase 2: Deep investigation

**Goal:** For every suspicious indicator found, understand what it is, how it got there, what it connects to, and what keeps it alive.

### For each suspicious process:
Determine its binary path, command-line arguments, working directory, environment variables, network connections, open file descriptors, parent/child relationships, and namespace membership. Extract any pool URLs, wallet addresses, C2 IPs, or authentication tokens from the command line or environment.

### For each suspicious file:
Determine its type, size, owner, timestamps, and content signatures. Check for known miner strings, packing (UPX), and architecture. Compute a hash for IOC reporting.

### Persistence — this is critical
Attackers almost always install persistence so their malware survives reboots and cleanup attempts. Check **all** of these vectors:

- **Scheduled tasks** — User crontabs (all users, not just root), system crontabs, cron directories, at jobs, systemd timers
- **Boot persistence** — Systemd services, SysV init scripts, rc.local, systemd generators
- **Login persistence** — Shell profile files (.bashrc, .profile, .bash_profile, .bash_login, .bash_logout) for all users, /etc/profile, /etc/profile.d/, /etc/bash.bashrc. Look for base64-encoded payloads, piped curl/wget, background execution, or disguised process names.
- **Library injection** — /etc/ld.so.preload, LD_PRELOAD in service environments, modified ld.so.conf
- **Kernel-level** — Loaded kernel modules (compare lsmod against /proc/modules to find hidden ones), modified kernel parameters
- **SSH access** — Unauthorized keys in authorized_keys files, SSH certificates, modified sshd_config
- **Filesystem tricks** — SUID/SGID binaries, immutable file attributes (chattr +i), bind mounts hiding files

### Lateral movement
Determine if the attacker can or has spread to other systems:
- SSH worm scripts that harvest keys/known_hosts/history
- Reverse shells or tunneling tools (gsocket, chisel, frp, ngrok, socat)
- Credentials exposed in shell history, environment files, or config files

### System integrity
Check whether the attacker has tampered with system binaries themselves. Use the OS package manager's verification feature if available (dpkg --verify, rpm -Va, apk verify). If critical tools like `ps`, `ss`, `ls`, or `netstat` are modified, your other findings may be unreliable — flag this immediately.

### Entry point
Try to determine how the attacker got in. Check:
- Auth logs for brute-force patterns or unauthorized access
- Web server logs for exploit signatures (command injection, path traversal, deserialization)
- Running web applications with known RCE vulnerabilities
- Exposed management panels, databases, or APIs without authentication
- Container escape indicators

---

## Phase 3: Report

**Goal:** Give the user a clear, structured picture of the compromise.

Present:
1. **What was found** — Each piece of malware with its type, location, owner, and purpose
2. **How it persists** — Every persistence mechanism
3. **Network IOCs** — All attacker IPs, domains, ports, and their roles (mining pool, C2, payload server)
4. **File IOCs** — Hashes and paths of all malicious files
5. **The attack chain** — How the attacker got in, what they deployed, and in what order (as far as you can reconstruct)
6. **Risk assessment** — Lateral movement potential, privilege escalation, data exposure, system integrity

If `--dry-run`, stop here.

Otherwise, ask: **"Shall I proceed with cleanup?"**

---

## Phase 4: Eradication

**Goal:** Remove all malware and persistence, block attacker infrastructure, and restore the system to a clean state.

**Order matters.** Kill watchdog/helper processes first so they can't respawn the miner. Then kill miners, backdoors, and worms. Then delete files, then remove persistence, then block IPs.

Principles:
- When cleaning shell profiles, remove only the malicious lines — don't destroy the user's legitimate configuration.
- When cleaning crontabs, remove only malicious entries unless the entire crontab is attacker-created.
- When blocking IPs, avoid duplicating existing firewall rules.
- When removing containers, stop before removing.
- When removing kernel modules, unload before deleting the .ko file.
- Document everything you remove so the user has a record.

---

## Phase 5: Verification

**Goal:** Confirm the system is clean.

Verify: no malicious processes remain, no malware files remain, no attacker network connections, CPU usage is normal, all persistence mechanisms are removed, firewall rules are in place. If any check fails, investigate and remediate before declaring clean.

---

## Phase 6: Recommendations

**Goal:** Help the user prevent reinfection.

Based on what you found, recommend specific actions:
- Which credentials need rotation and where they're stored
- What to patch and how (the specific vulnerable service/app)
- Which other servers to audit (based on SSH keys, known_hosts, worm targets)
- What to monitor going forward
- Whether reimaging is warranted (multiple attack waves, rootkit indicators, compromised system binaries, or inability to determine full scope)

---

## Reference: Patterns to recognize

These are **hints, not exhaustive lists.** Use your knowledge to identify malware beyond these patterns.

**Mining indicators:** Pool URLs (stratum+tcp://, pool domains like *moneroocean*, *2miners*, *nanopool*, *supportxmr*, *nicehash*), wallet addresses (long alphanumeric strings), algorithm names (randomx, cryptonight, ethash, kawpow), donation settings, hashrate references.

**Evasion techniques:** Process name spoofing (exec -a '[kthread]'), running from /dev/shm (tmpfs, no disk trace), deleted-binary execution, UPX packing, base64-encoded payloads in crontabs/profiles, disguising as legitimate services, using kernel thread bracket naming.

**Common hiding locations:** /tmp, /dev/shm, /var/tmp, /var/lock, /run, dotfiles/dotdirs in home directories (.config/, .local/, .cache/), /usr/lib/.*, /usr/share/.*, /opt/.*, /boot/.*.

**Companion malware:** Miners rarely come alone. Expect to find some combination of: watchdog processes, reverse shells, SSH worms, IoT botnet binaries, credential harvesters, privilege escalation exploits.

**Known botnet signatures (in strings output):** MIRAI, GAFGYT, TSUNAMI, MUHSTIK, KAITEN, BASHLITE, HAJIME — often appear alongside miner deployments on compromised servers.
