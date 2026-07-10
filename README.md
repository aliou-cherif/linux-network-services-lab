# Linux Network Services Lab

A virtual Linux lab for configuring, securing, testing, and documenting common network services using Rocky Linux Server and Ubuntu Client virtual machines.

## Project Objective

The goal of this project is to build a practical Linux-based network services environment and document the full configuration, testing, and troubleshooting process.

This project was created as a portfolio project to demonstrate practical skills in:

- Linux system administration
- Network service configuration
- Client-server testing
- Basic Linux security
- Firewall management
- Centralized logging
- Troubleshooting and documentation

## Lab Environment

| Machine | Role | Operating System | Hostname | IP Address |
|---|---|---|---|---|
| Rocky Server | Main network services server | Rocky Linux 9.8 | server.lelouch.org | 192.168.200.3/24 |
| Ubuntu Client | Testing client machine | Ubuntu 24.04.4 LTS | client.lelouch.org | 192.168.200.80/24 |

## Services Covered

This lab includes the following configurations:

- VirtualBox network setup
- Static IP configuration on Rocky Linux
- Hostname configuration
- DNS Server using BIND
- DHCP Server
- FTP / FTPS Server using vsftpd
- Mail Server using Postfix and Dovecot
- NFS Server
- SSH Server
- Apache Web Server with HTTPS
- Cowrie SSH Honeypot
- Firewall review using firewalld
- Centralized Syslog Server using rsyslog

## Repository Structure

```text
linux-network-services-lab/
├── README.md
├── docs/
│   ├── 01-lab-environment.md
│   ├── 02-dns-server.md
│   ├── 03-dhcp-server.md
│   ├── 04-ftp-ftps-server.md
│   ├── 05-mail-server.md
│   ├── 06-nfs-server.md
│   ├── 07-ssh-server.md
│   ├── 08-apache-web-server.md
│   ├── 09-cowrie-honeypot.md
│   ├── 10-firewall-review.md
│   └── 11-syslog-server.md
├── config/
│   ├── dns/
│   ├── dhcp/
│   ├── ftp/
│   ├── mail/
│   ├── nfs/
│   ├── ssh/
│   ├── apache/
│   ├── honeypot/
│   └── syslog/
└── screenshots/
    ├── lab-environment/
    ├── dns/
    ├── dhcp/
    ├── ftp/
    ├── mail/
    ├── nfs/
    ├── ssh/
    ├── apache/
    ├── honeypot/
    ├── firewall/
    └── syslog/
```

## Documentation

| Section | Description |
|---|---|
| [Lab Environment Setup](docs/01-lab-environment.md) | VirtualBox network setup, Rocky static IP, Ubuntu DHCP IP, hostname configuration, and connectivity testing |
| [DNS Server Configuration](docs/02-dns-server.md) | BIND DNS configuration with forward and reverse lookup zones for the `lelouch.org` domain |
| [DHCP Server Configuration](docs/03-dhcp-server.md) | DHCP server configuration on Rocky Linux with automatic IP assignment for the Ubuntu client |
| [FTP and FTPS Server Configuration](docs/04-ftp-ftps-server.md) | FTP server configuration using vsftpd, secured with SSL/TLS, and tested using FileZilla |
| [Mail Server Configuration](docs/05-mail-server.md) | Mail server configuration using Postfix and Dovecot, tested with Thunderbird |
| [NFS Server Configuration](docs/06-nfs-server.md) | NFS file sharing between Rocky Linux Server and Ubuntu Client, including permanent mount setup |
| [SSH Server Configuration](docs/07-ssh-server.md) | Remote administration using SSH, with root login disabled |
| [Apache Web Server Configuration](docs/08-apache-web-server.md) | Apache HTTP and HTTPS web hosting using a self-signed SSL/TLS certificate |
| [Cowrie SSH Honeypot Configuration](docs/09-cowrie-honeypot.md) | Cowrie SSH honeypot running on port `2222`, tested from Ubuntu, with captured logs |
| [Firewall Review with firewalld](docs/10-firewall-review.md) | Review of firewalld status, default zone, active interface, allowed services, and allowed ports |
| [Syslog Server Configuration](docs/11-syslog-server.md) | Centralized rsyslog server configuration, with Ubuntu forwarding logs to Rocky |

## Configuration Files

| Service | Configuration Folder |
|---|---|
| DNS | [config/dns](config/dns) |
| DHCP | [config/dhcp](config/dhcp) |
| FTP / FTPS | [config/ftp](config/ftp) |
| Mail | [config/mail](config/mail) |
| NFS | [config/nfs](config/nfs) |
| SSH | [config/ssh](config/ssh) |
| Apache | [config/apache](config/apache) |
| Cowrie Honeypot | [config/honeypot](config/honeypot) |
| Syslog | [config/syslog](config/syslog) |

## Security and Monitoring Features

This project includes several security-focused configurations.

| Feature | Purpose |
|---|---|
| SSH root login disabled | Prevents direct root login over SSH |
| FTPS | Secures FTP communication using SSL/TLS |
| HTTPS | Secures Apache web access using SSL/TLS |
| firewalld | Controls allowed services and ports on Rocky Linux |
| Cowrie Honeypot | Provides a fake SSH service to capture test login attempts and commands |
| Centralized Syslog | Collects Ubuntu client logs on the Rocky server for monitoring and analysis |

## Network Services Summary

| Service | Main Port(s) | Purpose |
|---|---|---|
| DNS | 53/tcp, 53/udp | Local name resolution for the lab domain |
| DHCP | 67/udp | Automatic IP assignment for the Ubuntu client |
| FTP / FTPS | 21/tcp, 990/tcp, 40000-40001/tcp | File transfer and secure file transfer |
| Mail | 25/tcp, 587/tcp, 993/tcp | SMTP, authenticated submission, and secure IMAP |
| NFS | NFS/RPC services | Shared directory access between Rocky and Ubuntu |
| SSH | 22/tcp | Remote server administration |
| Apache HTTP/HTTPS | 80/tcp, 443/tcp | Web hosting |
| Cowrie Honeypot | 2222/tcp | Fake SSH honeypot service |
| Syslog | 514/tcp, 514/udp | Centralized log forwarding and collection |

## Project Workflow

The project was built step by step.

Each service was configured, tested from the Ubuntu client where applicable, documented with screenshots, and supported with configuration files.

The general workflow was:

```text
Install service
Configure service
Open required firewall ports
Test from Ubuntu client
Verify logs or service output
Document configuration and screenshots
Upload configuration files to GitHub
```

## Troubleshooting Approach

Troubleshooting was included throughout the project.

Examples include:

- Fixing DNS resolution issues
- Adjusting DHCP configuration
- Configuring FTPS passive ports
- Correcting mail server settings
- Verifying NFS permissions and mounts
- Separating real SSH from the Cowrie honeypot
- Validating rsyslog configuration before restarting the service
- Checking firewall rules and listening ports

This helped make the project more realistic because Linux system administration often requires testing, fixing, and verifying configurations.

## Project Status

Completed for current scope.

Possible future improvements:

- Linux ACL permissions lab
- LDAP authentication
- Docker service deployment
- Basic monitoring dashboard
- Log analysis with security tools
- More advanced firewall restrictions
