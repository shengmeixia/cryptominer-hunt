# cryptominer-hunt

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey)](https://github.com)
[![Security](https://img.shields.io/badge/Security-Incident%20Response-red)](https://github.com)

An AI-powered skill for detecting, investigating, and removing cryptocurrency miners and associated malware from Linux servers.

## Why

Cryptominer malware is one of the most common payloads deployed on compromised Linux servers. Cleaning it up properly requires chasing a full attack chain — miners, droppers, persistence mechanisms, entry points, and lateral movement — across dozens of hiding spots. This is tedious, error-prone, and easy to do incompletely.

**cryptominer-hunt** turns an AI coding assistant into an incident responder. Rather than giving it a rigid checklist, the skill defines *goals and principles*, letting the AI adapt its approach to each unique server environment. It investigates like a human analyst would: following evidence, thinking adversarially, and chasing every thread to its end.

## What it does

| Phase | Goal |
|-------|------|
| **Triage** | Spot CPU anomalies, suspicious processes, unusual network activity, and files that shouldn't exist |
| **Deep investigation** | For every indicator: trace the binary, map persistence, check for lateral movement, verify system integrity, identify the entry point |
| **Report** | Structured findings — malware inventory, persistence mechanisms, network/file IOCs, reconstructed attack chain, risk assessment |
| **Eradication** | Kill processes (watchdogs first), remove files, clean persistence, block C2 infrastructure |
| **Verification** | Confirm the system is clean across all dimensions |
| **Recommendations** | Credential rotation, patching, lateral audit targets, monitoring, reimage assessment |

## Installation

Copy the skill file into your AI coding assistant's skills directory:

```bash
# Create the skill directory
mkdir -p ~/.claude/skills/cryptominer-hunt

# Copy the skill file
cp SKILL.md ~/.claude/skills/cryptominer-hunt/SKILL.md
```

Or clone this repo directly:

```bash
git clone https://github.com/shengmeixia/cryptominer-hunt.git ~/.claude/skills/cryptominer-hunt
```

## Usage

From a terminal session on a Linux server:

```
# Full scan — investigate and clean (with confirmation before destructive actions)
/cryptominer-hunt

# Dry run — investigate and report only, no modifications
/cryptominer-hunt --dry-run
```

### When to use it

- Server has unexplained high CPU usage
- You suspect a compromise or have confirmed one
- After an incident, to verify cleanup was thorough
- Routine security audit of internet-facing servers

## Design philosophy

This skill is intentionally **goal-oriented, not prescriptive**. It defines *what* to investigate, not *how*.

- **Zero hardcoded commands.** The AI chooses the right tools for the OS and environment it finds itself on.
- **Evidence-driven.** Each finding triggers deeper investigation — a miner implies a dropper, a dropper implies an entry point, an entry point implies persistence.
- **Adversarial thinking.** The skill prompts the AI to consider evasion techniques, hidden processes, tampered system binaries, and attack patterns it might otherwise overlook.
- **Parallel execution.** Independent checks run simultaneously for faster triage.

This means the skill works on Debian, Ubuntu, RHEL, Alpine, Arch, and any other Linux distribution without modification.

## Requirements

- An AI coding assistant that supports custom skills
- Linux server (any distribution)
- Root or sudo access on the target server

## Real-world tested

This skill was developed and refined during active incident response on a production server that had been compromised through a Next.js RCE vulnerability. The investigation uncovered:

- Multiple XMRig miner deployments targeting Monero mining pools
- IoT botnet binaries (Mirai/Gafgyt variants)
- Three rotating C2 servers delivering payloads via netcat
- Persistence through shell profile injection and cron jobs
- UPX-packed ELF binaries deployed to /tmp, /dev/shm, and application directories

The skill's design evolved through three iterations during that incident, moving from a rigid command checklist to the adaptive, goal-oriented framework you see today.

## License

[MIT](LICENSE)

## Contributing

Contributions are welcome! If you've used this skill in an incident and found patterns or hiding spots it missed, please open an issue or PR.
