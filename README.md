# Linux Network Services Lab

A virtual Linux lab for configuring and testing common network services using Rocky Linux Server and Ubuntu Client virtual machines.

## Project Objective

The goal of this project is to build a small Linux-based network services environment and document the configuration, testing, and troubleshooting process.

This project is designed as a practical portfolio project for Linux system administration, networking, and security fundamentals.

## Lab Environment

| Machine | Role | Operating System | Hostname | IP Address |
|---|---|---|---|---|
| Rocky Server | Main network services server | Rocky Linux 9.8 | server.lelouch.org | 192.168.200.3/24 |
| Ubuntu Client | Testing client machine | Ubuntu 24.04.4 LTS | client.lelouch.org | 192.168.200.80/24 |

## Services Covered

- Virtual machine network setup
- Static IP configuration on the Rocky Linux server
- Hostname configuration
- DNS Server using BIND
- DHCP Server
- FTP / FTPS Server using vsftpd
- Mail Server using Postfix and Dovecot
- NFS Server
- Apache Web Server
- Honeypot
- SSH Server
- Firewall rules
- Logs and troubleshooting

## Repository Structure

```text
linux-network-services-lab/
├── README.md
├── docs/
├── config/
└── screenshots/
```

## Documentation

| Section | Description |
|---|---|
| [Lab Environment Setup](docs/01-lab-environment.md) | VirtualBox network setup, server static IP, client dynamic IP, hostname configuration and connectivity test |
| [DNS Server Configuration](docs/02-dns-server.md) | BIND DNS configuration with forward and reverse lookup zones |
| [DHCP Server Configuration](docs/03-dhcp-server.md) | DHCP server configuration on Rocky Linux with automatic IP assignment for Ubuntu |
| [FTP and FTPS Server Configuration](docs/04-ftp-ftps-server.md) | FTP server configuration using vsftpd, secured with SSL/TLS and tested using FileZilla |
| [Mail Server Configuration](docs/05-mail-server.md) | Mail server configuration using Postfix and Dovecot, tested with Thunderbird |

## Configuration Files

| Service | Configuration Folder |
|---|---|
| DNS | [config/dns](config/dns) |
| DHCP | [config/dhcp](config/dhcp) |
| FTP / FTPS | [config/ftp](config/ftp) |
| Mail | [config/mail](config/mail) |

## Project Status

In progress.

Completed sections:

- Lab environment setup
- DNS server configuration
- DHCP server configuration
- FTP / FTPS server configuration
- Mail server configuration

Next section:

- NFS server configuration
