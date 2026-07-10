# Cowrie SSH Honeypot Configuration

## Objective

The objective of this section is to configure a Cowrie SSH honeypot on the Rocky Linux server and verify that it can capture SSH connection attempts from the Ubuntu client.

A honeypot is a decoy system used to attract, monitor, and record suspicious activity. Instead of giving access to the real server, the honeypot provides a fake environment where login attempts and commands can be safely logged.

In this lab, Cowrie was configured as a fake SSH service.

The real SSH service remains available on port `22`, while Cowrie listens on port `2222`.

```text
Real SSH service: 192.168.200.3:22
Cowrie honeypot SSH: 192.168.200.3:2222
```

This separation is important because it allows normal server administration through the real SSH service while keeping the honeypot isolated on a different port.

## Lab Information

| Machine | Role | Hostname | IP Address |
|---|---|---|---|
| Rocky Server | Cowrie Honeypot Server | server.lelouch.org | 192.168.200.3 |
| Ubuntu Client | Test Attacker / SSH Client | client.lelouch.org | 192.168.200.80 |

## Honeypot Overview

The honeypot flow is:

```text
Ubuntu Client → SSH connection on port 2222 → Cowrie Honeypot → Logs captured on Rocky
```

In simple terms:

1. Cowrie runs on the Rocky Linux server.
2. Cowrie listens on TCP port `2222`.
3. The Ubuntu client connects to the Rocky server using SSH on port `2222`.
4. Cowrie presents a fake Linux shell.
5. Commands entered by the client are captured in Cowrie logs.
6. The logs are reviewed on the Rocky server.

This setup is useful for learning how honeypots collect attacker behavior, such as usernames, passwords, source IP addresses, and commands.

## Configuration File

The important Cowrie configuration file is stored in the `config/honeypot/` folder.

| File | Purpose |
|---|---|
| [cowrie.cfg](../config/honeypot/cowrie.cfg) | Main Cowrie honeypot configuration used in this lab |

The active configuration used for this lab was:

```conf
[honeypot]
hostname = server

[ssh]
enabled = true
listen_endpoints = tcp:2222:interface=0.0.0.0
```

Explanation:

| Setting | Purpose |
|---|---|
| `[honeypot]` | Defines general honeypot settings |
| `hostname = server` | Sets the fake hostname displayed by the honeypot |
| `[ssh]` | Defines the fake SSH service configuration |
| `enabled = true` | Enables the Cowrie SSH honeypot service |
| `listen_endpoints = tcp:2222:interface=0.0.0.0` | Makes Cowrie listen on port `2222` on all network interfaces |

## Dependency Installation

The required packages were installed on Rocky Linux.

```bash
sudo dnf install -y git python3 python3-pip python3-devel gcc openssl-devel libffi-devel make
```

These packages are required to download Cowrie, create the Python environment, and build the Python dependencies used by Cowrie.

| Package | Purpose |
|---|---|
| `git` | Used to clone the Cowrie repository |
| `python3` | Python runtime |
| `python3-pip` | Python package installer |
| `python3-devel` | Python development files |
| `gcc` | Compiler used by some Python dependencies |
| `openssl-devel` | SSL/TLS development libraries |
| `libffi-devel` | Required by some security-related Python modules |
| `make` | Build tool used during dependency installation |

![Honeypot dependencies installed](../screenshots/honeypot/honeypot-dependencies-installed.png)

## Dedicated Cowrie User

A dedicated Linux user named `cowrie` was created.

```bash
sudo useradd -m -s /bin/bash cowrie
sudo passwd cowrie
id cowrie
ls -ld /home/cowrie
```

Cowrie was not installed or executed as `root`.

Using a dedicated user is better because it separates the honeypot files and processes from the main administrator account.

This also reduces risk if the honeypot service is misconfigured.

![Cowrie user created](../screenshots/honeypot/honeypot-cowrie-user-created.png)

## Cowrie Repository Download

The Cowrie source code was downloaded from GitHub using the dedicated `cowrie` user.

```bash
su - cowrie
git clone https://github.com/cowrie/cowrie.git
ls -l
ls -l cowrie
```

This created the Cowrie project directory at:

```text
/home/cowrie/cowrie
```

The repository contains the Cowrie source code, documentation, configuration directory, and startup tools.

![Cowrie repository cloned](../screenshots/honeypot/honeypot-cowrie-repository-cloned.png)

## Python Virtual Environment Installation

Cowrie requires a compatible Python environment.

A Python virtual environment was created inside the Cowrie directory.

```bash
cd ~/cowrie
python3.11 -m venv cowrie-env
source cowrie-env/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install --upgrade -e .
```

The virtual environment keeps Cowrie dependencies isolated from the system Python environment.

After installation, the environment was verified using:

```bash
python --version
pip --version
which cowrie
cowrie --help
```

The output confirmed that:

- Python 3.11 was used
- `pip` was working inside the virtual environment
- The `cowrie` command was available
- Cowrie was installed successfully

![Cowrie Python virtual environment install](../screenshots/honeypot/honeypot-python-virtualenv-install.png)

## Cowrie Configuration

The Cowrie configuration file was created in:

```text
/home/cowrie/cowrie/etc/cowrie.cfg
```

The configuration enabled the SSH honeypot and set Cowrie to listen on port `2222`.

```conf
[honeypot]
hostname = server

[ssh]
enabled = true
listen_endpoints = tcp:2222:interface=0.0.0.0
```

The port `2222` was used to avoid conflict with the real SSH service running on port `22`.

This means:

```text
Port 22   → real SSH administration
Port 2222 → Cowrie fake SSH honeypot
```

![Cowrie configuration](../screenshots/honeypot/honeypot-cowrie-config.png)

## Cowrie Service Start and Listening Port

Cowrie was started from the Python virtual environment.

```bash
cowrie start
cowrie status
ss -tulpn | grep 2222
```

The status output confirmed that Cowrie was running.

The `ss` command confirmed that port `2222` was listening on all interfaces:

```text
0.0.0.0:2222
```

This means the Ubuntu client can connect to Cowrie through the lab network.

![Cowrie service and port listening](../screenshots/honeypot/honeypot-cowrie-service-and-port-listening.png)

## Firewall Configuration

The Rocky Linux firewall was configured to allow inbound connections to Cowrie on port `2222/tcp`.

```bash
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

| Port | Service | Purpose |
|---|---|---|
| 2222/tcp | Cowrie SSH Honeypot | Allows the Ubuntu client to connect to the fake SSH service |

The firewall output confirmed that `2222/tcp` was allowed.

![Cowrie firewall port 2222](../screenshots/honeypot/honeypot-firewall-port-2222.png)

## Ubuntu SSH Honeypot Test

From the Ubuntu client, an SSH connection was made to the Rocky server on port `2222`.

```bash
ssh admin@192.168.200.3 -p 2222
```

A fake username was used for the test.

After connecting, the client was placed inside the Cowrie fake shell environment.

Several commands were executed:

```bash
ls
whoami
pwd
ls -l
```

The terminal showed a fake Linux environment. This confirms that the connection was handled by Cowrie and not by the real SSH service.

![Ubuntu SSH honeypot test](../screenshots/honeypot/honeypot-ubuntu-ssh-test.png)

## Cowrie Log Verification

After the Ubuntu test, the Cowrie log file was checked on the Rocky server.

```bash
tail -n 40 var/log/cowrie/cowrie.log
```

The log showed the SSH connection from the Ubuntu client.

Important information captured included:

- Source IP address: `192.168.200.80`
- Destination IP address: `192.168.200.3`
- Destination port: `2222`
- Username used during the test
- Login attempt
- Commands executed in the fake shell

The log showed commands such as:

```text
ls
whoami
pwd
ls -l
exit
```

This confirms that Cowrie successfully captured the SSH session activity.

![Cowrie log captured](../screenshots/honeypot/honeypot-cowrie-log-captured.png)

## Cowrie JSON Log Verification

Cowrie also stores structured logs in JSON format.

```bash
tail -n 10 var/log/cowrie/cowrie.json
```

The JSON log is useful because it records events in a structured format that can be parsed by security tools.

The JSON output included fields such as:

| Field | Meaning |
|---|---|
| `src_ip` | Source IP address of the connecting client |
| `dst_ip` | Destination IP address of the honeypot server |
| `dst_port` | Destination port used for the connection |
| `eventid` | Type of Cowrie event |
| `input` | Command entered by the connected user |
| `timestamp` | Time of the event |

This makes Cowrie useful for security monitoring and log analysis.

![Cowrie JSON log](../screenshots/honeypot/honeypot-cowrie-json-log.png)

## Troubleshooting

During the Cowrie installation, several real configuration issues were encountered and resolved.

### Issue 1: `python3-virtualenv` package was not available

The package `python3-virtualenv` was not available from the enabled Rocky Linux repositories.

Instead of relying on that package, the Python virtual environment was created using:

```bash
python3.11 -m venv cowrie-env
```

This solved the issue because Python can create virtual environments directly using the built-in `venv` module.

### Issue 2: Cowrie required Python 3.10 or newer

The default Python version on Rocky was Python 3.9.

Cowrie required a newer Python version, so the installation failed with Python 3.9.

The issue was solved by installing Python 3.11:

```bash
sudo dnf install -y python3.11 python3.11-pip python3.11-devel
```

The old virtual environment was removed and recreated using Python 3.11:

```bash
rm -rf cowrie-env
python3.11 -m venv cowrie-env
source cowrie-env/bin/activate
```

After that, Cowrie installed correctly.

### Issue 3: Older Cowrie commands did not match the current installation

Some older tutorials use commands such as:

```bash
bin/cowrie start
```

or refer to:

```text
etc/cowrie.cfg.dist
```

In this setup, Cowrie was installed inside the Python virtual environment using:

```bash
python -m pip install --upgrade -e .
```

After installation, the correct command was:

```bash
cowrie start
cowrie status
```

This confirmed that Cowrie was properly installed in the virtual environment.

## Useful Commands

Useful Cowrie management commands:

```bash
source cowrie-env/bin/activate
cowrie start
cowrie status
cowrie stop
cowrie restart
```

Useful log commands:

```bash
tail -n 40 var/log/cowrie/cowrie.log
tail -n 10 var/log/cowrie/cowrie.json
```

Useful network and firewall checks:

```bash
ss -tulpn | grep 2222
sudo firewall-cmd --list-all
```

## Security Notes

This honeypot was configured only inside the local VirtualBox lab network.

It was not exposed to the public Internet.

The purpose of this setup is educational:

- To understand how honeypots work
- To observe fake SSH login behavior
- To capture commands entered by a test client
- To learn how security logs can support monitoring and analysis

No real passwords should be used during honeypot testing.

## Result

Cowrie SSH honeypot was successfully installed and configured on Rocky Linux.

The honeypot was configured to listen on port `2222`, while the real SSH service remained on port `22`.

The Ubuntu client successfully connected to the fake SSH service, entered commands, and Cowrie captured the session activity in both standard log and JSON log formats.

This confirms that the Cowrie honeypot is working correctly in the local lab environment.
