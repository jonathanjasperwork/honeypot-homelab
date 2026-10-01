# Cowrie Homelab

## Overview

This project is a home-lab deployment of Cowrie, an SSH/Telnet honeypot designed to observe and record automated and interactive activity from hosts attempting to access the system.

The purpose of this project is to collect real-world attack telemetry and analyze the behavior of systems interacting with the honeypot.

The collected data is stored in PostgreSQL and analyzed using Python scripts.

## Objectives

The main objectives of this homelab are:

- Observe SSH login attempts.
- Identify commonly used usernames and passwords.
- Track attacker sessions and their duration.
- Identify source IP addresses.
- Collect SSH client information and versions.
- Record commands entered by attackers.
- Identify files downloaded by attackers.
- Record download URLs and file hashes.
- Geolocate observed source IP addresses.
- Analyze recurring attacker behavior.

## Architecture
                    Internet
                       |
                       v
                +--------------+
                |    Cowrie    |
                |   Honeypot   |
                +--------------+
                       |
                       | JSON logs
                       v
                +--------------+
                | Python       |
                | Log Importer |
                +--------------+
                       |
                       v
                +--------------+
                | PostgreSQL   |
                |   Database   |
                +--------------+
                       |
                       v
                +--------------+
                | Python       |
                | Report Tool  |
                +--------------+
                       |
                       v
                +--------------+
                |   Reports    |
                | .txt files   |
                +--------------+

## Data Collection

Cowrie generates JSONL event logs. The logs contain individual events associated with attacker sessions for 30 days (raw log will not be published)

Example log files:

cowrie.json.2026-09-22
cowrie.json.2026-09-23
cowrie.json.2026-09-24
cowrie.json.2026-09-25
...

# Project Structure
honeypot-lab/
│
├── log/
│   ├── cowrie.json.2026-09-22
│   ├── cowrie.json.2026-09-23
│   ├── cowrie.json.2026-09-24
│   └── ...
│
├── scripts/
│   ├── createTable.py
│   ├── insertTable.py
│   └── report.py
│
├── documents/
│   ├── username.txt
│   ├── password.txt
│   ├── username-password.txt
│   ├── ip-per-session.txt
│   ├── geo-ip.txt
│   ├── version.txt
│   ├── input.txt
│   ├── downloads.txt
│   └── session-duration.txt
│
└── README.md

# Environment

DB_HOST={YOUR_DB_HOSTNAME}
DB_PORT={YOUR_DB_PORT}
DB_NAME={YOUR_DB_NAME}
DB_USER={YOUR_DB_USER}
DB_PASSWORD={YOUR_DB_PASSWORD}

# Log Import Process

The importer processes all Cowrie JSON log files in the log directory.

Each line is parsed as an individual JSON event.

The event type is then used to determine which database table should receive the data.
```
Cowrie JSON log
      |
      v
Parse JSON event
      |
      v
Read eventid
      |
      +---- session.connect ------> sessions
      |
      +---- session.closed -------> sessions
      |
      +---- login.success ---------> auth
      |
      +---- login.failed ----------> auth
      |
      +---- client.version --------> clients
      |
      +---- command.input ---------> input
      |
      +---- command.success -------> commands
      |
      +---- file_download ----------> downloads
      |
      +---- session.params ---------> params
      |
      +---- log.closed ------------> ttylog
```

## Disclaimer

This project is intended for security research, monitoring, and educational purposes within a controlled honeypot environment.

The data represents activity observed by the honeypot and should be interpreted within the limitations of honeypot telemetry, IP geolocation, and automated Internet scanning.