# Linux Network Services Lab

A virtual Linux lab for configuring DNS, DHCP, FTP, NFS, SSH, Apache, Mail, Firewall and Honeypot services.

## Project Objective

The goal of this project is to configure and test essential Linux network services in a virtual environment using Rocky Linux Server and Ubuntu Client virtual machines.

## Services Covered

- Static IP configuration
- Hostname configuration
- Users, groups, permissions and ownership
- DNS Server
- DHCP Server
- FTP / FTPS Server
- NFS Server
- Apache Web Server
- Mail Server
- Honeypot
- SSH Server
- Firewall rules
- ACL
- System logs and troubleshooting

## Lab Environment

| Machine | Role | Operating System | IP Address |
|---|---|---|---|
| Server | Main network services server | Rocky Linux | To be added |
| Client | Testing client machine | Ubuntu Desktop | To be added |

## Documentation

| Section | Description |
|---|---|
| [Lab Environment Setup](docs/01-lab-environment.md) | Initial VM setup, static IP addressing and connectivity test |
| [DNS Server Configuration](docs/02-dns-server.md) | BIND DNS server configuration with forward and reverse lookup zones |
