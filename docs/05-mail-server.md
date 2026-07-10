# Mail Server Configuration

## Objective

The objective of this section is to configure a local mail server on Rocky Linux using Postfix and Dovecot.

This mail server allows local Linux users to send and receive emails inside the lab environment. The setup uses:

- **Postfix** for SMTP mail delivery
- **Dovecot** for IMAP/IMAPS mailbox access
- **BIND DNS** for mail-related DNS records
- **OpenSSL** for TLS certificate generation
- **Thunderbird** as the mail client on Ubuntu

The final setup was tested from an Ubuntu client using Thunderbird.

## Lab Information

| Machine | Role | Hostname | IP Address |
|---|---|---|---|
| Rocky Server | Mail Server | server.lelouch.org | 192.168.200.3 |
| Ubuntu Client | Mail Client | client.lelouch.org | 192.168.200.80 |

## Mail Server Architecture

The mail server was configured using two main components: Postfix and Dovecot.

Postfix is responsible for sending and delivering mail. When a user sends an email, Postfix handles the SMTP process and delivers the message to the correct local mailbox.

Dovecot is responsible for allowing mail clients to access those mailboxes. In this project, Thunderbird connects to Dovecot using IMAPS to read the user's emails securely.

The mail flow in this lab is:

```text
Thunderbird → Postfix → Maildir mailbox → Dovecot → Thunderbird
```

In simple terms:

1. Thunderbird sends an email using SMTP submission.
2. Postfix receives and delivers the email.
3. The email is stored inside the recipient user's `Maildir/` mailbox.
4. Dovecot allows Thunderbird to read the mailbox using IMAPS.

## Mail Users

This lab uses existing Linux users as mail users.

| Linux User | Email Address |
|---|---|
| lelouch | lelouch@lelouch.org |
| robin | robin@lelouch.org |

No separate virtual mail user database was created. The mail addresses are based on local Linux system accounts.

## Mail Services Used

| Service | Purpose |
|---|---|
| Postfix | SMTP server used for sending and delivering mail |
| Dovecot | IMAP/IMAPS server used for reading mailboxes |
| BIND DNS | Provides A and MX records for the mail domain |
| OpenSSL | Generates the self-signed TLS certificate |
| Thunderbird | Mail client used for testing send and receive operations |

## Configuration Files

The important configuration files are stored in the `config/mail/` folder.

| File | Purpose |
|---|---|
| [postfix-main.cf](../config/mail/postfix-main.cf) | Main Postfix mail server configuration |
| [postfix-master.cf](../config/mail/postfix-master.cf) | Postfix service configuration for SMTP and submission port |
| [dovecot-10-mail.conf](../config/mail/dovecot-10-mail.conf) | Dovecot Maildir location configuration |
| [dovecot-10-auth.conf](../config/mail/dovecot-10-auth.conf) | Dovecot authentication configuration |
| [dovecot-10-ssl.conf](../config/mail/dovecot-10-ssl.conf) | Dovecot SSL/TLS configuration |
| [dovecot-10-master.conf](../config/mail/dovecot-10-master.conf) | Dovecot authentication socket for Postfix |

Only the important active configuration lines are stored in GitHub. The original system configuration files contain many comments and default settings, so the GitHub versions are cleaned configuration excerpts.

## DNS Mail Records

The DNS forward zone was updated to include an MX record for the domain and an A record for the mail server.

```dns
@       IN      MX      10 mail.lelouch.org.
mail    IN      A       192.168.200.3
```

The `A` record maps `mail.lelouch.org` to the Rocky server IP address.

The `MX` record tells the domain `lelouch.org` which mail server should handle email for the domain.

In this lab:

```text
lelouch.org uses mail.lelouch.org as its mail server
mail.lelouch.org points to 192.168.200.3
```

![Mail DNS zone file](../screenshots/mail/mail-dns-zone-file.png)

The DNS zone was validated using `named-checkzone`, and the MX/A records were tested using `nslookup`.

```bash
sudo named-checkzone lelouch.org /var/named/fwd.lelouch.org.db
sudo systemctl restart named
nslookup -type=mx lelouch.org 192.168.200.3
nslookup mail.lelouch.org 192.168.200.3
```

![Mail DNS MX record test](../screenshots/mail/mail-dns-mx-record-test.png)

## Package Installation

Postfix, Dovecot, and OpenSSL were installed on the Rocky Linux server.

```bash
sudo dnf install -y postfix dovecot openssl
rpm -q postfix dovecot openssl
```

Postfix provides the SMTP server, Dovecot provides the IMAP/IMAPS service, and OpenSSL is used to generate the TLS certificate.

![Mail packages installed](../screenshots/mail/mail-packages-installed.png)

## TLS Certificate Generation

A self-signed TLS certificate was generated for the mail server.

```bash
sudo mkdir -p /etc/ssl/mail

sudo openssl req -x509 -nodes -newkey rsa:2048 -days 365 \
-keyout /etc/ssl/mail/mailserver.key \
-out /etc/ssl/mail/mailserver.crt \
-subj "/C=MY/ST=KualaLumpur/L=KualaLumpur/O=LinuxLab/OU=Mail/CN=mail.lelouch.org"

sudo chmod 600 /etc/ssl/mail/mailserver.key
sudo ls -l /etc/ssl/mail/
```

The certificate is used to secure mail connections. Since this is a local lab, a self-signed certificate is acceptable.

The generated files are:

| File | Purpose |
|---|---|
| `mailserver.crt` | Public certificate used by the mail services |
| `mailserver.key` | Private key used by the server |

The private key was protected with `chmod 600` so that only root can read it.

![Mail TLS certificate generation](../screenshots/mail/mail-tls-certificate-generation-files.png)

> The private key file was not uploaded to GitHub for security reasons.

## Postfix Main Configuration

Postfix was configured with the domain `lelouch.org` and hostname `mail.lelouch.org`.

```conf
myhostname = mail.lelouch.org
mydomain = lelouch.org
myorigin = $mydomain

inet_interfaces = all
inet_protocols = ipv4

mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 127.0.0.0/8, 192.168.200.0/24

home_mailbox = Maildir/
```

Important settings:

| Setting | Explanation |
|---|---|
| `myhostname` | Defines the mail server hostname |
| `mydomain` | Defines the mail domain |
| `myorigin` | Sets the domain used for locally generated mail |
| `inet_interfaces = all` | Allows Postfix to listen on available network interfaces |
| `inet_protocols = ipv4` | Uses IPv4 for this lab |
| `mydestination` | Defines domains that this server accepts mail for |
| `mynetworks` | Allows trusted local network clients |
| `home_mailbox = Maildir/` | Stores mail in each user's `~/Maildir/` directory |

TLS was also enabled using the generated certificate.

```conf
smtpd_tls_cert_file = /etc/ssl/mail/mailserver.crt
smtpd_tls_key_file = /etc/ssl/mail/mailserver.key
smtpd_tls_security_level = may
```

The TLS configuration allows Postfix to support encrypted SMTP connections.

![Postfix main configuration 1](../screenshots/mail/mail-postfix-main-config-1.png)

![Postfix main configuration 2](../screenshots/mail/mail-postfix-main-config-2.png)

## Dovecot Mail Location

Dovecot was configured to use the Maildir format.

```conf
mail_location = maildir:~/Maildir
```

This tells Dovecot where user mailboxes are stored.

For example:

```text
/home/lelouch/Maildir/
/home/robin/Maildir/
```

Postfix delivers messages into the user's `Maildir/`, and Dovecot reads messages from the same location.

![Dovecot mail location configuration](../screenshots/mail/mail-dovecot-mail-location-config.png)

## Dovecot Authentication

Plaintext authentication was disabled unless SSL/TLS is used.

```conf
disable_plaintext_auth = yes
auth_mechanisms = plain login
```

The `plain` and `login` mechanisms allow Thunderbird to authenticate with a username and password.

Because SSL/TLS is required, the login credentials are not sent over an unencrypted connection.

![Dovecot authentication configuration](../screenshots/mail/mail-dovecot-auth-config.png)

## Dovecot SSL/TLS Configuration

Dovecot was configured to require SSL/TLS and use the generated mail certificate.

```conf
ssl = required
ssl_cert = </etc/ssl/mail/mailserver.crt
ssl_key = </etc/ssl/mail/mailserver.key
```

This configuration secures IMAP access through IMAPS on port `993`.

![Dovecot SSL configuration](../screenshots/mail/mail-dovecot-ssl-config.png)

## Postfix and Dovecot Authentication Socket

A Dovecot authentication socket was configured so that Postfix can authenticate users through Dovecot.

```conf
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
```

This creates a local Unix socket that Postfix can use to ask Dovecot to verify usernames and passwords.

The permission `0660` limits access to the socket, making the configuration cleaner and safer than allowing access to everyone.

![Dovecot Postfix authentication socket](../screenshots/mail/mail-dovecot-postfix-auth-socket.png)

## Postfix SASL Configuration

Postfix was configured to use Dovecot for SMTP authentication.

```conf
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_tls_auth_only = yes
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
```

Important settings:

| Setting | Explanation |
|---|---|
| `smtpd_sasl_type = dovecot` | Tells Postfix to use Dovecot for authentication |
| `smtpd_sasl_path = private/auth` | Points Postfix to the Dovecot authentication socket |
| `smtpd_sasl_auth_enable = yes` | Enables SMTP authentication |
| `smtpd_tls_auth_only = yes` | Allows authentication only when TLS is used |
| `reject_unauth_destination` | Prevents the server from becoming an open relay |

This is important because a mail server should not allow unauthenticated users to send mail anywhere.

![Postfix Dovecot SASL configuration](../screenshots/mail/mail-postfix-dovecot-sasl-config.png)

## Service Status

Postfix and Dovecot were enabled, restarted, and verified.

```bash
sudo systemctl enable --now postfix dovecot
sudo systemctl restart postfix dovecot
sudo systemctl status postfix
sudo systemctl status dovecot
```

Both services were confirmed as active and running.

![Postfix service status](../screenshots/mail/mail-postfix-service-status.png)

![Dovecot service status](../screenshots/mail/mail-dovecot-service-status.png)

## SMTP Submission Port

The Postfix submission service was enabled on port `587`.

```conf
submission inet n       -       n       -       -       smtpd
```

Port `587` is used by mail clients such as Thunderbird to submit outgoing mail to the mail server.

The listening ports were verified using `ss`.

```bash
sudo ss -tulpn | grep master
```

![Postfix submission port configuration](../screenshots/mail/mail-postfix-submission-port-config.png)

## Firewall Configuration

The required mail ports were allowed through the firewall.

```bash
sudo firewall-cmd --permanent --add-port=25/tcp
sudo firewall-cmd --permanent --add-port=587/tcp
sudo firewall-cmd --permanent --add-port=993/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

| Port | Service | Purpose |
|---|---|---|
| 25/tcp | SMTP | Used by Postfix for SMTP mail delivery |
| 587/tcp | SMTP Submission | Used by Thunderbird to send mail through Postfix |
| 993/tcp | IMAPS | Used by Thunderbird to read mail securely through Dovecot |

For the final Thunderbird test, the most important ports were:

- `587/tcp` for sending mail
- `993/tcp` for reading mail

![Mail firewall rules](../screenshots/mail/mail-firewall-rules.png)

## Dovecot Listening Ports

Dovecot was verified using `ss`.

```bash
sudo ss -tulpn | grep dovecot
```

The important secure port for this project is `993/tcp`, which is used for IMAPS.

Other Dovecot ports may appear, such as IMAP or POP3 ports, but the final client test used secure IMAPS.

![Dovecot IMAPS port listening](../screenshots/mail/mail-dovecot-imaps-port-listening.png)

## Local Mail Delivery Test

A local mail delivery test was performed from the Rocky Linux server.

The test confirmed that mail sent to `robin@lelouch.org` was delivered to the `Maildir` mailbox.

```bash
cat <<EOF | /usr/sbin/sendmail -v robin@lelouch.org
From: lelouch@lelouch.org
To: robin@lelouch.org
Subject: Local mail test

This is a test email from lelouch to robin.
EOF

sudo ls -R /home/robin/Maildir | head -30
sudo ls -l /home/robin/Maildir/new
```

This test proves that Postfix can deliver mail locally to a Linux user mailbox.

![Mail local delivery test](../screenshots/mail/mail-local-delivery-test.png)

## Ubuntu DNS Mail Test

The Ubuntu client was able to resolve the mail server and MX record using the Rocky DNS server.

```bash
nslookup mail.lelouch.org 192.168.200.3
nslookup -type=mx lelouch.org 192.168.200.3
```

This confirms that the DNS records were configured correctly and that Ubuntu can query the Rocky DNS server for mail records.

![Ubuntu DNS MX test](../screenshots/mail/mail-ubuntu-dns-mx-test.png)

## Thunderbird Client Configuration

Thunderbird was configured on the Ubuntu client using the Rocky mail server IP address.

The DNS records for `mail.lelouch.org` were verified separately using `nslookup`, but the IP address was used in Thunderbird to avoid client-side name resolution issues during the final test.

| Setting | Value |
|---|---|
| Incoming protocol | IMAP |
| Incoming server | 192.168.200.3 |
| Incoming port | 993 |
| Incoming security | SSL/TLS |
| Outgoing server | 192.168.200.3 |
| Outgoing port | 587 |
| Outgoing security | STARTTLS |
| Authentication | Normal password |
| Username | robin |

![Thunderbird Robin account configuration](../screenshots/mail/mail-thunderbird-robin-account-config.png)

The account was created successfully, and Thunderbird received the existing local test emails.

![Thunderbird account created](../screenshots/mail/mail-thunderbird-robin-account-created.png)

## Final Send and Receive Test

A final test was performed using Thunderbird.

The test confirmed that:

- `lelouch@lelouch.org` can send mail to `robin@lelouch.org`
- `robin@lelouch.org` can reply to `lelouch@lelouch.org`
- Thunderbird can send mail through Postfix
- Thunderbird can read mail through Dovecot

![Thunderbird send and receive success](../screenshots/mail/mail-thunderbird-send-receive-success.png)

## Troubleshooting

During testing, Thunderbird initially failed to send mail because the outgoing SMTP server hostname was incorrect.

The outgoing server was corrected to use the Rocky mail server IP address:

```text
192.168.200.3
```

After updating the SMTP server settings, Thunderbird successfully sent and received mail.

Useful troubleshooting checks for this mail setup include:

```bash
sudo systemctl status postfix
sudo systemctl status dovecot
postconf -n
sudo doveconf -n
sudo ss -tulpn | grep master
sudo ss -tulpn | grep dovecot
sudo firewall-cmd --list-all
```

These commands help verify service status, active configuration, listening ports, and firewall rules.

## Result

The mail server was successfully configured using Postfix and Dovecot.

Postfix handled SMTP mail delivery, Dovecot provided IMAPS mailbox access, and Thunderbird was able to send and receive emails between local Linux users.

The final result confirms that the Rocky Linux mail server is working correctly in the local lab environment.
