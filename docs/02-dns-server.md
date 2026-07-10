# DNS Server Configuration

## Objective

The objective of this section is to configure a local DNS server using BIND on Rocky Linux.

The DNS server is used to resolve local hostnames inside the lab network. Instead of always using IP addresses such as `192.168.200.3`, the client can use names such as `server.lelouch.org`.

This makes the lab environment easier to manage and more realistic, because most real networks rely on DNS to translate hostnames into IP addresses.

## Lab Information

| Machine | Role | Hostname | IP Address |
|---|---|---|---|
| Rocky Server | DNS Server | server.lelouch.org | 192.168.200.3 |
| Ubuntu Client | DNS Client | client.lelouch.org | 192.168.200.4 |

> Note: During the DNS configuration test, the Ubuntu client used `192.168.200.4`. Later in the project, the DHCP server assigned the Ubuntu client a different address. This DNS section documents the original DNS testing stage.

## DNS Configuration Overview

BIND was installed and configured on the Rocky Linux server.

The DNS setup includes:

- Main BIND configuration file
- Forward lookup zone
- Reverse lookup zone
- Zone file validation
- Firewall rule for DNS traffic
- DNS testing from the Rocky server
- DNS testing from the Ubuntu client

The DNS server was configured for the local domain:

```text
lelouch.org
```

This allows local machines to resolve names such as:

```text
server.lelouch.org
client.lelouch.org
```

## How DNS Works in This Lab

In this lab, the Rocky Linux server acts as the DNS resolver for the local network.

The basic DNS flow is:

```text
Ubuntu Client → asks DNS Server → Rocky checks zone files → returns IP address
```

For example:

```text
Ubuntu asks: What is the IP address of server.lelouch.org?
Rocky DNS replies: 192.168.200.3
```

This is useful because services can be accessed using names instead of memorizing IP addresses.

## Forward and Reverse DNS

Two DNS zones were configured:

| Zone Type | Purpose |
|---|---|
| Forward lookup zone | Converts hostname to IP address |
| Reverse lookup zone | Converts IP address to hostname |

Example of forward lookup:

```text
server.lelouch.org → 192.168.200.3
```

Example of reverse lookup:

```text
192.168.200.3 → server.lelouch.org
```

Forward lookup is the most common DNS function. Reverse lookup is also useful for troubleshooting, logging, and verifying that IP addresses match the correct hostnames.

## Configuration Files

The DNS configuration files are available in the `config/dns/` folder.

| File | Purpose |
|---|---|
| [named.conf](../config/dns/named.conf) | Main BIND DNS configuration file |
| [fwd.lelouch.org.db](../config/dns/fwd.lelouch.org.db) | Forward lookup zone file |
| [rvs.200.168.192.db](../config/dns/rvs.200.168.192.db) | Reverse lookup zone file |

Only the important DNS configuration files were added to GitHub. These files show how the local domain and DNS records were configured.

## BIND Service Status

The `named` service is the BIND DNS service on Rocky Linux.

It was enabled and started on the Rocky Linux server.

```bash
sudo systemctl status named
```

The service status confirms that the DNS server is running.

![DNS named status](../screenshots/dns/dns-named-status.png)

## named.conf Zone Configuration

The main BIND configuration file was updated with two DNS zones:

```text
lelouch.org
200.168.192.in-addr.arpa
```

The `lelouch.org` zone is used for forward DNS resolution.

The `200.168.192.in-addr.arpa` zone is used for reverse DNS resolution for the `192.168.200.0/24` network.

![DNS named.conf zones](../screenshots/dns/dns-named-conf-zones.png)

## Forward Lookup Zone

The forward lookup zone maps hostnames to IP addresses.

Examples:

```text
server.lelouch.org → 192.168.200.3
client.lelouch.org → 192.168.200.4
```

This allows the client to reach the server using its fully qualified domain name instead of only using its IP address.

The forward zone file contains records such as:

| Record | Purpose |
|---|---|
| `SOA` | Defines the authority information for the zone |
| `NS` | Defines the name server for the domain |
| `A` | Maps a hostname to an IPv4 address |

![DNS forward zone file](../screenshots/dns/dns-forward-zone-file.png)

## Reverse Lookup Zone

The reverse lookup zone maps IP addresses back to hostnames.

Examples:

```text
192.168.200.3 → server.lelouch.org
192.168.200.4 → client.lelouch.org
```

Reverse DNS is useful when checking which hostname belongs to an IP address.

The reverse zone uses `PTR` records.

| Record | Purpose |
|---|---|
| `PTR` | Maps an IP address back to a hostname |

![DNS reverse zone file](../screenshots/dns/dns-reverse-zone-file.png)

## Zone Validation

The forward and reverse zone files were checked using `named-checkzone`.

```bash
named-checkzone lelouch.org /var/named/fwd.lelouch.org.db
named-checkzone 200.168.192.in-addr.arpa /var/named/rvs.200.168.192.db
```

Both zone files returned `OK`.

This confirms that the DNS zone files were valid and did not contain syntax errors.

![DNS zone validation](../screenshots/dns/dns-zone-validation.png)

## Firewall Configuration

DNS uses port `53`.

DNS commonly uses:

| Protocol | Purpose |
|---|---|
| UDP 53 | Standard DNS queries |
| TCP 53 | Larger DNS responses and zone-related operations |

The firewall was configured to allow DNS traffic over both UDP and TCP.

```bash
firewall-cmd --permanent --add-port=53/udp
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --reload
```

![DNS firewall port 53](../screenshots/dns/dns-firewall-port-53.png)

## Rocky Server DNS Settings

The Rocky Linux server was configured to use itself as one of its DNS resolvers.

```text
DNS Server: 192.168.200.3
```

This allows the Rocky server to test and resolve records from its own DNS service.

![Rocky DNS settings](../screenshots/dns/dns-rocky-dns-settings.png)

## DNS Testing on Rocky Server

DNS resolution was tested from the Rocky server using `nslookup`.

```bash
nslookup server.lelouch.org
nslookup client.lelouch.org
```

The Rocky server successfully resolved both local hostnames.

This proves that the DNS service was running correctly on the server side.

![Rocky DNS nslookup test](../screenshots/dns/dns-rocky-nslookup-test.png)

## Ubuntu Client DNS Configuration

The Ubuntu client was configured to use the Rocky Linux server as its DNS server.

The `resolvectl status` command confirmed that Ubuntu was using:

```text
DNS Server: 192.168.200.3
```

This means Ubuntu sends DNS queries to the Rocky DNS server.

![Ubuntu resolvectl DNS status](../screenshots/dns/dns-ubuntu-resolvectl.png)

## DNS Testing on Ubuntu Client

DNS resolution was tested from the Ubuntu client using `nslookup` and `ping`.

```bash
nslookup server.lelouch.org
nslookup client.lelouch.org
ping -c 2 server.lelouch.org
```

The Ubuntu client successfully resolved and reached the Rocky server using its fully qualified domain name.

This confirms that DNS resolution worked not only locally on Rocky, but also from another machine in the network.

![Ubuntu DNS test](../screenshots/dns/dns-ubuntu-nslookup-ping-test.png)

## Troubleshooting Checks

Useful DNS troubleshooting commands include:

```bash
sudo systemctl status named
sudo journalctl -u named
sudo named-checkconf
sudo named-checkzone lelouch.org /var/named/fwd.lelouch.org.db
sudo named-checkzone 200.168.192.in-addr.arpa /var/named/rvs.200.168.192.db
sudo firewall-cmd --list-all
nslookup server.lelouch.org 192.168.200.3
nslookup 192.168.200.3 192.168.200.3
```

These commands help verify:

- BIND service status
- DNS configuration syntax
- Zone file validity
- Firewall rules
- Forward DNS lookup
- Reverse DNS lookup

## Result

The DNS server was configured successfully using BIND on Rocky Linux.

The Rocky Linux server can resolve local DNS records, and the Ubuntu client can use the Rocky server as its DNS resolver.

Both forward and reverse DNS zone files were validated successfully, and the Ubuntu client was able to resolve and reach the Rocky server using its hostname.

This DNS setup became the foundation for later services in the lab, such as mail records, server hostnames, and client-server communication.
