# Lab Environment Setup

## Objective

The objective of this section is to prepare the virtual lab environment before configuring the main Linux network services.

This lab uses two virtual machines:

- Rocky Linux Server
- Ubuntu Client

The server will host the main network services, while the client will be used to test connectivity and service access.

## Virtual Machines

| Machine | Role | Operating System | IP Address |
|---|---|---|---|
| Server | Main network services server | Rocky Linux | To be added |
| Client | Testing client machine | Ubuntu Desktop | To be added |

## Network Configuration

Static IP addresses were configured on both virtual machines to ensure stable communication between the server and the client.

This is important because services such as DNS, DHCP, FTP, NFS, Apache, Mail and SSH require the server to have a fixed IP address.

## Connectivity Test

Connectivity between the two machines was tested using the `ping` command.

```bash
ping server-ip
ping client-ip
```

## Result

The server and client were able to communicate successfully using their static IP addresses.

This confirms that the basic network connection between the two virtual machines is working correctly.

## Notes

This section only covers the initial lab environment setup. The detailed configuration of each service will be documented in separate files.
