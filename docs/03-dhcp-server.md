# DHCP Server Configuration

## Objective

The objective of this section is to configure the Rocky Linux server as a DHCP server and verify that the Ubuntu client receives its IP address automatically from Rocky.

The DHCP server provides network configuration to the client, including:

- IP address
- Subnet mask
- Default gateway
- DNS server

## Lab Information

| Machine | Role | Hostname | IP Address |
|---|---|---|---|
| Rocky Server | DHCP Server | server.lelouch.org | 192.168.200.3 |
| Ubuntu Client | DHCP Client | client.lelouch.org | 192.168.200.80 |

## DHCP Configuration Overview

The DHCP service was installed and configured on the Rocky Linux server.

The Ubuntu client was then configured to receive its IP address automatically.

To make sure the Ubuntu client receives its IP address from Rocky and not from VirtualBox, the VirtualBox NAT Network DHCP service was disabled.

## DHCP Configuration File

The DHCP configuration file is available in the `config/dhcp/` folder.

| File | Purpose |
|---|---|
| [dhcpd.conf](../config/dhcp/dhcpd.conf) | Main DHCP server configuration file |

## Rocky Server Static IP Verification

Before configuring DHCP, the Rocky server IP address was verified.

The server must keep a static IP address because a DHCP server should not depend on an automatically assigned address.

```bash
ip addr
```

![Rocky static IP](../screenshots/dhcp/dhcp-rocky-static-ip.png)

## DHCP Scope Configuration

The DHCP scope was configured in:

```text
/etc/dhcp/dhcpd.conf
```

The DHCP server was configured with the following values:

| Setting | Value |
|---|---|
| Network | 192.168.200.0/24 |
| DHCP Range | 192.168.200.80 - 192.168.200.90 |
| Default Gateway | 192.168.200.1 |
| Subnet Mask | 255.255.255.0 |
| DNS Server | 192.168.200.3 |
| Default Lease Time | 3600 seconds |
| Maximum Lease Time | 86400 seconds |

```conf
default-lease-time 3600;
max-lease-time 86400;
authoritative;

subnet 192.168.200.0 netmask 255.255.255.0 {
    range 192.168.200.80 192.168.200.90;
    option routers 192.168.200.1;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 192.168.200.3;
}
```

![DHCP configuration file](../screenshots/dhcp/dhcp-config-file.png)

## DHCP Service Status

The DHCP service was enabled and started on the Rocky Linux server.

```bash
sudo systemctl enable --now dhcpd.service
sudo systemctl status dhcpd
```

The service status shows that `dhcpd` is enabled and running.

![DHCP service status](../screenshots/dhcp/dhcp-service-status.png)

## Firewall Configuration

The firewall was configured to allow DHCP traffic.

```bash
sudo firewall-cmd --permanent --add-service=dhcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

The firewall output shows that the DHCP service is allowed.

![DHCP firewall service](../screenshots/dhcp/dhcp-firewall-service.png)

## VirtualBox DHCP Disabled

The VirtualBox NAT Network DHCP service was disabled.

This step is important because the Ubuntu client must receive its IP address from the Rocky DHCP server instead of the VirtualBox DHCP service.

![VirtualBox DHCP disabled](../screenshots/dhcp/dhcp-virtualbox-dhcp-disabled.png)

## Ubuntu Client DHCP Verification

After disabling the VirtualBox DHCP service and starting the Rocky DHCP server, the Ubuntu client received an IP address automatically.

```bash
nmcli
```

The Ubuntu client received:

| Setting | Value |
|---|---|
| IP Address | 192.168.200.80/24 |
| Gateway | 192.168.200.1 |
| DNS Server | 192.168.200.3 |
| Interface | enp0s3 |

![Ubuntu DHCP IP assigned](../screenshots/dhcp/dhcp-ubuntu-ip-assigned.png)

## DHCP Lease Verification

The DHCP lease file on Rocky confirms that the Ubuntu client received an IP address from the DHCP server.

```bash
sudo cat /var/lib/dhcpd/dhcpd.leases
```

The lease file shows that `192.168.200.80` was assigned to the client.

![DHCP lease file](../screenshots/dhcp/dhcp-lease-file.png)

## Connectivity Test

The Ubuntu client tested connectivity by pinging the Rocky server using its FQDN.

```bash
ping -c 4 server.lelouch.org
```

The ping test was successful with 0% packet loss.

![Ubuntu ping server](../screenshots/dhcp/dhcp-ubuntu-ping-server.png)

## Internet Connectivity Test

Internet connectivity was also tested from the Ubuntu client using a web browser.

![DHCP internet test](../screenshots/dhcp/dhcp-internet-test.png)

## Important Note

After DHCP was configured, the Ubuntu client received a new IP address from the DHCP range.

The previous DNS record for `client.lelouch.org` may still point to the old IP address `192.168.200.4`.

For this reason, the main connectivity test in this section uses:

```text
server.lelouch.org
```

The server keeps the same static IP address, so its DNS record remains valid.

## Result

The DHCP server was configured successfully.

The Ubuntu client received an IP address automatically from the Rocky Linux DHCP server.

The client also received the correct gateway and DNS server information, and it was able to reach the Rocky server using its domain name.
