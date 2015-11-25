+++
date = "2015-11-25T12:00:00Z"
title = "How to run a mail server"
+++

## The Why
As a result of the recent public discussion regarding online privacy I decided
to take a few measures to give myself some more privacy online. One such
measure was the creation of a new [4096 bit PGP public key](https://pgp.mit.edu/pks/lookup?op=vindex&search=0x7BE4075E24AB8942)
for myself. I also started running my DNS requests through tor for some more
privacy, which was inspired by Drew's excellent [blog post](https://drewdevault.com/2015/11/11/Bring-more-tor-into-your-life.html).

Lastly, I decided it was time to stop letting Google handle all the mail for my
domain. This was partly because it's been costing me roughly the price of a
[pint of beer in London](http://www.telegraph.co.uk/finance/newsbysector/retailandconsumer/11541536/Is-London-the-most-expensive-city-in-which-to-buy-a-pint.html)
each month, and partly because it seemed like it would be a ~~fun~~ learning
experience.

As a note to self, and to spare you the ~~fun~~ part of the learning experience
I thought I'd share the details of my set up to show how straightforward it can
be to get a solid mail server online.

## Overview

For my mail server there are three key pieces of software at play:

 - [postfix](http://www.postfix.org) - Provides an SMTP server
 - [dovecot](http://www.dovecot.org) - Handled mailboxes, authentication, and IMAP
 - [MariaDB](https://mariadb.org/) - SQL database of email accounts

Inbound mail is given to postfix via SMTP. Postfix then checks against the SQL
database whether the email address should be accepted, and performs spam
filtering. If the email passes postfix's checks it's passed to dovecot over a
unix socket via LMTP. Dovecot is then responsible for placing the mail in the
correct user's mail directory.

Email are accessed over IMAP using dovecot. Upon connection, users are
authenticated by dovecot against accounts in the SQL database, and on success
are connected to their mail directory.

Outbound mail is also given to postfix, however this time postfix requires
authentication. Postfix sends the login details to dovecot over a unix pipe,
which checks against the SQL database whether the user is permitted to send
mail. If so, postfix is given the approval to do so, and relays the email to
the relevant remote SMTP server.

## Configuring MariaDB

Create a new database. I called mine `maildb`. Also create a new MariaDB user,
`mail` with read-only access to `maildb`.

Within `maildb` you want to create the following three tables:

```sql
CREATE TABLE `aliases` (
  `mail` varchar(120) NOT NULL,
  `destination` varchar(120) NOT NULL,
  UNIQUE KEY `mail` (`mail`)
);

CREATE TABLE `domains` (
  `domain` varchar(120) NOT NULL
);

CREATE TABLE `users` (
  `email` varchar(128) NOT NULL,
  `crypt` varchar(128) NOT NULL,
  PRIMARY KEY (`email`),
  UNIQUE KEY `id` (`email`)
);
```

### Aliases Table
The purpose of `aliases` is for an email address mapping step performed by
postfix. Every email that is received locally has its address transformed using
this lookup table.

To receive email at an address it is **required** to be covered by this table.
I have `harry@exec64.co.uk` mapped to `harry@exec64.co.uk`, which passes my
mail through without changing the destination.

However, if you're the administrator of the server, you'll want to receive mail
for users such as `postmaster@example.com`, so you'd need to map that to
`your_address@example.com`.

You can also provide a catch-all address for a domain by excluding the name
portion of the email address. In my case that would be mapping `@exec64.co.uk`
to `harry@exec64.co.uk.`

### Domains Table

This table is even simpler to configure. It should simply contains all the
domains for which your server should receive mail for. It's used by postfix to
determine whether to send mail to another server, or to pass it to dovecot.

In my case it simply contains `exec64.co.uk`.

### Users Table
This is the most important table, which is used solely by dovecot to authenticate
users. The `email` column should contain the users full email address, and the
`crypt` column should contain an encrypted form of their password.

Here's an example query to insert a new user:

```sql
INSERT INTO users (email,crypt) VALUES ('user@example.com',encrypt('plaintext_password', CONCAT('$5$', MD5(RAND()))) );
```

## Configuring Dovecot

Now that the SQL database has been set up correctly, the next step is to set up
Dovecot to authenticate users, manage their email directories, and provide an
IMAP server for users to retrieve their mail from.

The recommended way to configure dovecot seems to be to have a main configuration
file at `/etc/dovecot/dovecot.conf` which includes many other configuration files
from `/etc/dovecot/conf.d/` but I found that to be an overcomplication for my
needs. Simply having one relatively short config file was perfectly adequate.

### Create the virtual user

Traditionally email addresses were tied to unix users on servers, and the mail
was stored in each user's home directory. In our case, we want our email
addresses to be independent of the users on the system, so instead all the mail
will be "owned" by a virtual user named `virtual`. (Forgive me, I don't study
creative writing.)

To create the user and the mail directory run the following commands:
```bash
sudo mkdir /var/spool/mail/virtual
sudo groupadd --system virtual -g 5000
sudo useradd --system virtual -u 5000 -g 5000
sudo chown -R virtual:virtual /var/spool/mail/virtual
```

Now we're ready to configure dovecot itself. It's all fairly straight forward.
Here's a copy of my configuration file. Everything ought to be fairly self
explanatory, and if it isn't, dovecot has some pretty good
[documentation](http://wiki2.dovecot.org/).

### /etc/dovecot/dovecot.conf

```ini
#provide acesss to imap and ltmp
protocols = imap lmtp

#don't handle mail without SSL silly
#if you don't have a cert go to LetsEncrypt.org
ssl = yes
ssl_cert = </path/to/fullchain.pem
ssl_key = </path/to/privkey.pem

listen = *, ::

base_dir = /var/run/dovecot/

instance_name = dovecot

login_greeting = Dovecot ready.

login_trusted_networks = 127.0.0.1

disable_plaintext_auth = no

auth_mechanisms = plain login

#Mail is stored in "Maildir" in each user's home directory
mail_location = maildir:~/Maildir

# authenticate users using SQL
passdb {
  driver = sql
  #this config file is provided beneath
  args = /etc/dovecot/dovecot-sql.conf.ext
}

userdb {
  driver = static
  #store each user's mail in /var/spool/mail/virtual/name/domain
  #i.e. virtual/harry/exec64.co.uk
  args = uid=virtual gid=virtual home=/var/spool/mail/virtual/%d/%n
}

namespace inbox {
  inbox = yes
}

#the mail is owned by the user/group named "virtual"
mail_uid = virtual
mail_gid = virtual

auth_mechanisms = plain login

#provide an authentication service to postfix
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
}

protocol lmtp {
  #this option is required, or you won't be able to receive mail
  postmaster_address = postmaster@exec64.co.uk
}

#accept delivery from postfix via lmtp
service lmtp {
  unix_listener /var/spool/postfix/private/dovecot-lmtp {
    mode = 0600
    user = postfix
    group = postfix
  }
}
```

### /etc/dovecot/dovecot-sql.conf.ext

```ini
driver = mysql
connect = host=localhost dbname=maildb user=mail password=mailpasswordhere
default_pass_scheme = SHA256-CRYPT #make sure this matches the encryption scheme you use

password_query = SELECT email as user, crypt as password FROM users WHERE email = '%u';
```

### Testing dovecot

At this point, you should now have a working IMAP server that you can connect to
and retrieve mail from. Of course you won't have any mail to read, but you should
still verify that you can now connect, and also check that the relevant mail
directory was created in `/var/spool/mail/virtual` for your mail.

## Configuring Postfix

And now for the final step, providing an SMTP server to send/receive mail with.

In my case, I only needed to set up one main configuration file for postfix,
and a couple of other files to allow it to query the SQL database.

Please note that this configuration doesn't provide any spam filtering.

Again, if something doesn't make sense to you, [RTFM](http://www.postfix.org/BASIC_CONFIGURATION_README.html).

### /etc/postfix/main.cf

```ini
#We don't need to set the domain here, it's configured by the virtual mailboxes
mydomain = localhost
myhostname = $mydomain
myorigin = $mydomain
mydestination = localhost
mynetworks_style = host
relay_domains = $mydestination
relayhost =
smtpd_banner = $myhostname ESMTP

#virtual mailbox settings
virtual_mailbox_base = /var/spool/mail/virtual
virtual_mailbox_maps = mysql:/etc/postfix/sql_mailbox.cf
virtual_alias_maps = mysql:/etc/postfix/sql_alias.cf
virtual_mailbox_domains = mysql:/etc/postfix/sql_domains.cf
#uid of virtual user is 5000
virtual_uid_maps = static:5000
virtual_gid_maps = static:5000
#tell postfix to send mail for virtual inboxes to dovecot
virtual_transport = lmtp:unix:private/dovecot-lmtp

#aliases
alias_maps = hash:/etc/postfix/aliases
alias_database = hash:/etc/postfix/aliases

#tls parameters
smtpd_tls_cert_file=/path/to/fullchain.pem
smtpd_tls_key_file=/path/to/privkey.pem
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_loglevel = 2
smtpd_tls_received_header = yes
smtpd_tls_session_cache_timeout = 3600s
tls_random_source = dev:/dev/urandom

#authentication settings
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
smtpd_recipient_limit = 16
smtpd_soft_error_limit = 3
smtpd_hard_error_limit = 12
smtpd_helo_restrictions = permit_mynetworks, warn_if_reject reject_non_fqdn_hostname, reject_invalid_hostname, permit
smtpd_sender_restrictions = permit_mynetworks, warn_if_reject reject_non_fqdn_sender, reject_unknown_sender_domain, reject_unauth_pipelining, permit
smtpd_client_restrictions = reject_rbl_client sbl.spamhaus.org, reject_rbl_client blackholes.easynet.nl
smtpd_data_restrictions = reject_unauth_pipelining
smtpd_helo_required = yes
smtpd_delay_reject = yes
disable_vrfy_command = yes

#other
compatibility_level = 2
queue_directory = /var/spool/postfix
command_directory = /usr/bin
daemon_directory = /usr/lib/postfix/bin
data_directory = /var/lib/postfix
mail_owner = postfix
inet_interfaces = all
local_recipient_maps =
sendmail_path = /usr/bin/sendmail
newaliases_path = /usr/bin/newaliases
mailq_path = /usr/bin/mailq
setgid_group = postdrop
html_directory = no
manpage_directory = /usr/share/man
sample_directory = /etc/postfix
readme_directory = /usr/share/doc/postfix
inet_protocols = ipv4
meta_directory = /etc/postfix
shlib_directory = /usr/lib/postfix
delay_warning_time = 1h
unknown_local_recipient_reject_code = 450
maximal_queue_lifetime = 7d
minimal_backoff_time = 1000s
maximal_backoff_time = 8000s
smtp_helo_timeout = 60s
```

We also need to tell postfix how to query the information it needs from the
SQL database.

### /etc/postfix/sql_alias.cf
```ini
user=mail
password=password
hosts=127.0.0.1
dbname=maildb
query = select destination from aliases where mail = '%s'
```

### /etc/postfix/sql_mailbox.cf
```ini
user=mail
password=password
hosts=127.0.0.1
dbname=maildb
query = select 1 from users where email = '%s'
```

### /etc/postfix/sql_domains.cf
```ini
user=mail
password=password
hosts=127.0.0.1
dbname=maildb
query = select 1 from domains where domain = '%'
```

## DNS

The final step to get postfix working is to tell the world where it is. To do
this you need to set a MX DNS record. The purpose of an MX record is to indicate
where a domain's mail server is hosted. While most DNS records should resolve to
IP addresses, for MX records you should be returning another domain name. This
is to allow resolving that domain to either an IPv4 or an IPv6 address.

MX records also have a `priority` field. With a single record it's irrelevant so
you can just set it to `10`. For the address, give the domain name of the server
that postfix is running on.

## Final Thoughts

Congratulations, you should now have your own mail server.

You'll probably want to add spam filtering, and a few other goodies. There's a
good guide to setting that up [here](http://flurdy.com/docs/postfix/).

Have any questions or corrections? Leave a comment below, or drop me an email.
