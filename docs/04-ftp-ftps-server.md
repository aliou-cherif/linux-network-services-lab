# FTP and FTPS Server Configuration

## Objective

The objective of this section is to configure an FTP server using `vsftpd` on Rocky Linux, then secure the service using SSL/TLS.

FTP is used to transfer files between a client and a server. However, basic FTP is not secure because usernames, passwords, and transferred data can be sent in clear text.

To improve security, the FTP service was upgraded to FTPS. FTPS uses SSL/TLS encryption to protect authentication and file transfers.

In this lab:

- Rocky Linux acts as the FTP/FTPS server
- Ubuntu acts as the FTP/FTPS client
- FileZilla is used to test the secure connection and file transfer

## Lab Information

| Machine | Role | Hostname | IP Address |
|---|---|---|---|
| Rocky Server | FTP / FTPS Server | server.lelouch.org | 192.168.200.3 |
| Ubuntu Client | FTP / FTPS Client | client.lelouch.org | 192.168.200.80 |

## FTP and FTPS Overview

The service was configured in two main stages.

First, a basic FTP server was configured using `vsftpd`. This allowed local Linux users to connect and transfer files.

Then, SSL/TLS was enabled to secure the FTP service. This changed the setup from plain FTP to FTPS.

The final connection flow is:

```text
FileZilla on Ubuntu → FTPS connection → vsftpd on Rocky → user FTP directory
```

## FTP vs FTPS

| Protocol | Description |
|---|---|
| FTP | Standard file transfer protocol. It is functional but not encrypted. |
| FTPS | FTP secured with SSL/TLS encryption. It protects login credentials and file transfers. |

In this project, the final test was performed using **explicit FTP over TLS** from FileZilla.

## Configuration Overview

This section includes:

- Installing and enabling `vsftpd`
- Configuring local user access
- Disabling anonymous FTP access
- Restricting local users using chroot
- Opening FTP firewall rules
- Generating a self-signed SSL/TLS certificate
- Enabling FTPS in `vsftpd`
- Configuring passive mode ports
- Opening FTPS and passive mode firewall ports
- Testing secure FTPS access using FileZilla

## Configuration File

The active FTP/FTPS configuration is available in the `config/ftp/` folder.

| File | Purpose |
|---|---|
| [vsftpd.conf](../config/ftp/vsftpd.conf) | Main FTP / FTPS server configuration file |

Only the important active configuration lines were added to GitHub. The original `vsftpd.conf` file contains many comments and default settings, so the GitHub version is a cleaned configuration excerpt.

## FTP Service Installation and Status

The `vsftpd` package was installed and the service was enabled on the Rocky Linux server.

```bash
sudo dnf install vsftpd
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo systemctl status vsftpd
```

The service status confirms that the FTP server is installed and running.

![FTP vsftpd status](../screenshots/ftp/ftp-vsftpd-status.png)

## Basic FTP Configuration

Anonymous FTP access was disabled, while local user login and write access were enabled.

```conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
```

Explanation:

| Setting | Purpose |
|---|---|
| `anonymous_enable=NO` | Disables anonymous FTP login |
| `local_enable=YES` | Allows local Linux users to log in |
| `write_enable=YES` | Allows authenticated users to upload or modify files |

Disabling anonymous access is important because the server should only allow known local users to connect.

![FTP local users write config](../screenshots/ftp/ftp-local-users-write-config.png)

## Chroot Configuration

Local users were restricted to their FTP environment using chroot settings.

```conf
chroot_local_user=YES
chroot_list_enable=YES
allow_writeable_chroot=YES
chroot_list_file=/etc/vsftpd/chroot_list
ls_recurse_enable=YES
```

The purpose of chroot is to prevent FTP users from browsing the entire Linux filesystem.

With chroot enabled, users are restricted to their FTP directory instead of being able to move freely through system directories.

Important settings:

| Setting | Purpose |
|---|---|
| `chroot_local_user=YES` | Restricts local users to their FTP directory |
| `chroot_list_enable=YES` | Enables the use of a chroot list file |
| `allow_writeable_chroot=YES` | Allows users to write inside their restricted directory |
| `chroot_list_file=/etc/vsftpd/chroot_list` | Defines the chroot list file path |
| `ls_recurse_enable=YES` | Allows recursive directory listing |

![FTP chroot config](../screenshots/ftp/ftp-chroot-config.png)

## Additional FTP Settings

The FTP server was configured with a local root directory and other service settings.

```conf
local_root=public_html
use_localtime=YES
seccomp_sandbox=NO
```

Explanation:

| Setting | Purpose |
|---|---|
| `local_root=public_html` | Sets the FTP starting directory for local users |
| `use_localtime=YES` | Displays file times using local server time |
| `seccomp_sandbox=NO` | Disables seccomp sandboxing to avoid compatibility issues in this lab |

The `local_root` setting makes the FTP environment cleaner because users are directed to a specific folder instead of starting directly in the full home directory.

![FTP basic config](../screenshots/ftp/ftp-basic-config.png)

## FTP Firewall Rules

The FTP service was allowed through the Rocky Linux firewall.

```bash
sudo firewall-cmd --add-service=ftp
sudo firewall-cmd --runtime-to-permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

The FTP service uses port `21/tcp` for the control connection.

Opening the FTP service allows the Ubuntu client to connect to the Rocky FTP server.

![FTP firewall rules](../screenshots/ftp/ftp-firewall-rules.png)

## FTPS Certificate Generation

OpenSSL was used to generate a self-signed certificate for `vsftpd`.

```bash
sudo dnf install openssl
sudo mkdir /etc/ssl/vsftpd
sudo openssl req -x509 -nodes -keyout /etc/ssl/vsftpd/vsftpd-selfsigned.pem -out /etc/ssl/vsftpd/vsftpd-selfsigned.pem -days 365 -newkey rsa:2048
```

The certificate is used by `vsftpd` to encrypt FTP connections with SSL/TLS.

Because this is a local lab environment, a self-signed certificate is acceptable. In a production environment, a certificate from a trusted certificate authority would normally be used.

![FTPS certificate generation](../screenshots/ftp/ftps-openssl-certificate-generation.png)

## Certificate File Verification

The certificate file was created inside `/etc/ssl/vsftpd/`.

```bash
sudo ls -l /etc/ssl/vsftpd/
```

In this lab, the certificate and private key were stored in the same `.pem` file:

```text
/etc/ssl/vsftpd/vsftpd-selfsigned.pem
```

![FTPS certificate file](../screenshots/ftp/ftps-certificate-file.png)

> The certificate/private key file was not uploaded to GitHub for security reasons.

## SSL/TLS Configuration

SSL/TLS was enabled in the `vsftpd` configuration file.

```conf
ssl_enable=YES
ssl_tlsv1_2=YES
ssl_sslv2=NO
ssl_sslv3=NO

rsa_cert_file=/etc/ssl/vsftpd/vsftpd-selfsigned.pem
rsa_private_key_file=/etc/ssl/vsftpd/vsftpd-selfsigned.pem
```

Explanation:

| Setting | Purpose |
|---|---|
| `ssl_enable=YES` | Enables SSL/TLS support in vsftpd |
| `ssl_tlsv1_2=YES` | Allows TLS 1.2 |
| `ssl_sslv2=NO` | Disables old SSLv2 |
| `ssl_sslv3=NO` | Disables old SSLv3 |
| `rsa_cert_file` | Defines the certificate file used by vsftpd |
| `rsa_private_key_file` | Defines the private key file used by vsftpd |

Old SSL versions were disabled because they are outdated and insecure.

![FTPS SSL TLS config](../screenshots/ftp/ftps-ssl-tls-config.png)

## FTPS Security and Passive Mode Configuration

The server was configured to force SSL/TLS for local user logins and data transfers.

```conf
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_ciphers=HIGH
require_ssl_reuse=NO

pasv_min_port=40000
pasv_max_port=40001
debug_ssl=YES
```

Important settings:

| Setting | Purpose |
|---|---|
| `allow_anon_ssl=NO` | Prevents anonymous SSL login |
| `force_local_data_ssl=YES` | Forces encrypted file transfer for local users |
| `force_local_logins_ssl=YES` | Forces encrypted login for local users |
| `ssl_ciphers=HIGH` | Uses stronger SSL/TLS cipher suites |
| `require_ssl_reuse=NO` | Improves compatibility with clients such as FileZilla |
| `pasv_min_port=40000` | Defines the first passive mode port |
| `pasv_max_port=40001` | Defines the last passive mode port |
| `debug_ssl=YES` | Enables SSL debugging information |

Passive mode ports are important because FTP uses separate connections for control and data transfer. By defining a small passive port range, the firewall can be configured more precisely.

![FTPS security passive config](../screenshots/ftp/ftps-security-passive-config.png)

## FTPS Service Status

After updating the configuration, the `vsftpd` service was restarted successfully.

```bash
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

This confirms that the FTPS configuration did not break the service.

![FTPS service status](../screenshots/ftp/ftps-service-status.png)

## FTPS Firewall Rules

The firewall was updated to allow FTPS and passive mode ports.

```bash
sudo firewall-cmd --permanent --add-port=990/tcp
sudo firewall-cmd --permanent --add-port=40000-40001/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

| Port | Purpose |
|---|---|
| 21/tcp | FTP control connection / explicit FTPS start |
| 990/tcp | FTPS-related secure port |
| 40000-40001/tcp | Passive mode data transfer ports |

The passive mode ports are required so that FileZilla can complete directory listing and file transfers.

![FTPS firewall rules](../screenshots/ftp/ftps-firewall-rules.png)

## FileZilla FTPS Client Configuration

FileZilla was configured on the Ubuntu client to connect to the Rocky Linux server using explicit FTP over TLS.

| Setting | Value |
|---|---|
| Protocol | FTP - File Transfer Protocol |
| Host | 192.168.200.3 |
| Encryption | Require explicit FTP over TLS |
| Logon type | Ask for password |
| User | ChSNA |

The server IP address was used directly to keep the client configuration simple and avoid DNS-related issues during the FTPS test.

![FTPS FileZilla site manager](../screenshots/ftp/ftps-filezilla-site-manager.png)

## FTPS Connection Test

FileZilla successfully established a TLS connection to the Rocky Linux server.

The connection log shows:

```text
Initializing TLS...
TLS connection established.
Logged in
Directory listing successful
```

This confirms that:

- FileZilla reached the Rocky server
- TLS negotiation succeeded
- The user authenticated successfully
- The FTP directory listing was accessible

![FTPS FileZilla connection success](../screenshots/ftp/ftps-filezilla-connection-success.png)

## FTPS File Transfer Test

A file transfer was tested successfully using FileZilla over FTPS.

This final test confirms that the server was not only reachable, but also able to transfer files securely.

![FTPS FileZilla transfer test](../screenshots/ftp/ftps-filezilla-transfer-test.png)

## Troubleshooting Checks

Useful troubleshooting commands for this FTP/FTPS setup include:

```bash
sudo systemctl status vsftpd
sudo journalctl -u vsftpd
sudo firewall-cmd --list-all
sudo ss -tulpn | grep vsftpd
sudo grep -v '^\s*#' /etc/vsftpd/vsftpd.conf | sed '/^\s*$/d'
```

These commands help verify:

- `vsftpd` service status
- FTP/FTPS logs
- Firewall rules
- Listening ports
- Active vsftpd configuration

Common issues to check:

| Issue | Possible Cause |
|---|---|
| FileZilla cannot connect | Firewall rule missing or vsftpd not running |
| Login fails | Wrong username/password or local user access disabled |
| Directory listing fails | Passive ports not opened |
| TLS warning appears | Self-signed certificate being used |
| Upload fails | `write_enable` disabled or directory permissions incorrect |

## Result

The FTP server was configured successfully using `vsftpd`.

The service was then secured using SSL/TLS, converting the setup into FTPS.

FileZilla confirmed that a TLS connection could be established from the Ubuntu client, and the final file transfer test was successful.

This confirms that FTPS was working correctly in the local lab environment.
