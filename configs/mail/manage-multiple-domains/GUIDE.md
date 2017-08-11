NOTE:
Using lookup table type is hash
https://www.howtoforge.com/linux_postfix_virtual_hosting
```
disable selinux -> reboot
hostnamectl set-hostname mail.lab.global.com
yum --enablerepo=centosplus install postfix -y
systemctl enable postfix
yum install net-tools telnet mailx -y
```

/etc/postfix/main.cf
```
# domain list
virtual_mailbox_domains = /etc/postfix/vhosts.txt
# base of mail boxes
virtual_mailbox_base = /var/spool/vmail
# map between email address to mailbox location
virtual_mailbox_maps = hash:/etc/postfix/vmaps.txt
virtual_uid_maps = static:1000
virtual_gid_maps = static:1000
virtual_alias_maps = hash:/etc/postfix/valias.txt
```
/etc/postfix/vhosts.txt
```
abc.com
xyz.vn
tele.scope
```
/etc/postfix/vmaps.txt
If you specify a / at the end of the location, it becomes Maildir format. If not, it is mbox
```
joe@abc.com abc.com/joe/
joe@xyz.vn xyz.vn/joe/
john@tele.scope tele.scope/john
```

convert to hash file:
postmap /etc/postfix/vmaps.txt
-> output /etc/postfix/vmaps.txt.db

create mailbox
```
mkdir -p /var/spool/vmail/abc.com/joe/{cur,new,tmp}
mkdir -p /var/spool/vmail/xyz.vn/joe/{cur,new,tmp}
mkdir -p /var/spool/vmail/tele.scope/john/{cur,new,tmp}
chmod 700 -R /var/spool/vmail/
```

create virtual user for access all mailbox
```
groupadd -g 1000 virtual
useradd -u 1000 -g virtual -d /var/spool/vmail/ -s /sbin/nologin -c "Virtual Mail User" virtual
chown -R virtual:virtual /var/spool/vmail/
```

/etc/postfix/valias.txt
```
joe@abc.com joe@yahoo.com
```

yum -y install dovecot

cd /etc/dovecot
cp dovecot.conf dovecot.conf.orig
dovecot.conf
```
protocols = imap pop3
ssl = required
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key = </etc/pki/dovecot/private/dovecot.pem
log_path = /var/log/dovecot
info_log_path = /var/log/dovecot.info
mail_location = maildir:/var/spool/vmail/%d/%n

### username login must in format:
### joe@abc.com -> abc.com map %d, joe map %n
### dovecot need to know mailbox that postfix save mail. Dovecot will read
### mail from mailbox.

auth_mechanisms = plain digest-md5
# Authentication for passwd-file users. Included from 10-auth.conf.
#
# passwd-like file with specified location.
# <doc/wiki/AuthDatabase.PasswdFile.txt>

passdb {
  driver = passwd-file
  # %u is full username eg: joe@abc.com
  # %n is joe
  # %d is abc.com
  args = scheme=CRYPT username_format=%u /etc/dovecot/users
}

userdb {
  driver = passwd-file
  args = username_format=%u /etc/dovecot/users

  # Default fields that can be overridden by passwd-file
  #default_fields = quota_rule=*:storage=1G

  # Override fields from passwd-file
  #override_fields = home=/home/virtual/%u
}
```

openssl passwd -1 -salt abc 123
$1$abc$98/EDagBiz63dxD3fhRFk1

/etc/dovecot/users
```
joe@abc.com:$1$abc$98/EDagBiz63dxD3fhRFk1:1000:1000::/var/spool/vmail/abc.com/
```
login as joe@abc.com/123 but then dovecot use user 1000:1000 (virtual:virtual) to
access mailbox

systemctl restart dovecot
systemctl restart postfix

[root@mail spool]# telnet localhost 110
Trying ::1...
Connected to localhost.
Escape character is '^]'.
+OK Dovecot ready.
user joe@abc.com
+OK
pass 123
+OK Logged in.