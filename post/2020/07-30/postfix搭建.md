
[TOC]

### 删除sendmail相关服务（如果有）

```bash
service sendmail stop
apt remove sendmail
```


### 安装postfix
使用ubuntu操作系统，直接参考文档 https://wiki.debian.org/PostfixAndSASL



###  /etc/postfix/main.cf
```ini
# See /usr/share/postfix/main.cf.dist for a commented, more complete version
  

# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# See http://www.postfix.org/COMPATIBILITY_README.html -- default to 2 on
# fresh installs.
compatibility_level = 2

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = smtp.mydomain.com
mydomain = mydomain.com
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = $myhostname, mydomain.com, host_name, localhost.localdomain, localhost
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all




smtpd_sasl_path = smtpd
smtp_use_tls = no
smtpd_sasl_authenticated_header = yes
smtpd_sasl_local_domain = $myhostname
smtpd_sasl_auth_enable = yes
broken_sasl_auth_clients = yes
smtp_tls_security_level = encrypt
smtp_sasl_tls_security_options = noanonymous
smtpd_recipient_restrictions = permit_sasl_authenticated, permit_mynetworks, reject_unauth_destination



# DKIM
milter_default_action = accept
milter_protocol = 2
smtpd_milters = inet:localhost:8892
non_smtpd_milters = inet:localhost:8892

```

### /etc/default/saslauthd-postfix
```ini
#
# Settings for saslauthd daemon
# Please read /usr/share/doc/sasl2-bin/README.Debian for details.
#

# Should saslauthd run automatically on startup? (default: no)
START=yes

# Description of this saslauthd instance. Recommended.
# (suggestion: SASL Authentication Daemon)
DESC="SASL Authentication Daemon for postfox"

# Short name of this saslauthd instance. Strongly recommended.
# (suggestion: saslauthd)
NAME="saslauthd-postf"

# Which authentication mechanisms should saslauthd use? (default: pam)
#
# Available options in this Debian package:
# getpwent  -- use the getpwent() library function
# kerberos5 -- use Kerberos 5
# pam       -- use PAM
# rimap     -- use a remote IMAP server
# shadow    -- use the local shadow password file
# sasldb    -- use the local sasldb database file
# ldap      -- use LDAP (configuration is in /etc/saslauthd.conf)
#
# Only one option may be used at a time. See the saslauthd man page
# for more information.
#
# Example: MECHANISMS="pam" 这个选项决定了账户是linux账户
MECHANISMS="shadow"

# Additional options for this mechanism. (default: none)
# See the saslauthd man page for information about mech-specific options.
MECH_OPTIONS=""

# How many saslauthd processes should we run? (default: 5)
# A value of 0 will fork a new process for each connection.
THREADS=5

# Other options (default: -c -m /var/run/saslauthd)
# Note: You MUST specify the -m option or saslauthd won't run!
#
# WARNING: DO NOT SPECIFY THE -d OPTION.
# The -d option will cause saslauthd to run in the foreground instead of as
# a daemon. This will PREVENT YOUR SYSTEM FROM BOOTING PROPERLY. If you wish
# to run saslauthd in debug mode, please run it by hand to be safe.
#
# See /usr/share/doc/sasl2-bin/README.Debian for Debian-specific information.
# See the saslauthd man page and the output of 'saslauthd -h' for general
# information about these options.
#
# Example for chroot Postfix users: "-c -m /var/spool/postfix/var/run/saslauthd"
# Example for non-chroot Postfix users: "-c -m /var/run/saslauthd"
#
# To know if your Postfix is running chroot, check /etc/postfix/master.cf.
# If it has the line "smtp inet n - y - - smtpd" or "smtp inet n - - - - smtpd"
# then your Postfix is running in a chroot.
# If it has the line "smtp inet n - n - - smtpd" then your Postfix is NOT
# running in a chroot.
OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"

```

### /etc/opendkim.conf 

```ini

# This is a basic configuration that can easily be adapted to suit a standard
# installation. For more advanced options, see opendkim.conf(5) and/or
# /usr/share/doc/opendkim/examples/opendkim.conf.sample.

# Log to syslog
Syslog                  yes
# Required to use local socket with MTAs that access the socket as a non-
# privileged user (e.g. Postfix)
UMask                   007

# Sign for example.com with key in /etc/dkimkeys/dkim.key using
# selector '2007' (e.g. 2007._domainkey.example.com)
Domain                  mydomain.com
KeyFile         /etc/mail/dkim_mykey.key
Selector                myselector

# Commonly-used options; the commented-out versions show the defaults.
#Canonicalization       simple
#Mode                   sv
#SubDomains             no

# Socket smtp://localhost
#
# ##  Socket socketspec
# ##
# ##  Names the socket where this filter should listen for milter connections
# ##  from the MTA.  Required.  Should be in one of these forms:
# ##
# ##  inet:port@address           to listen on a specific interface
# ##  inet:port                   to listen on all interfaces
# ##  local:/path/to/socket       to listen on a UNIX domain socket
#
Socket                  inet:8892@localhost
#Socket                 local:/var/run/opendkim/opendkim.sock

##  PidFile filename
###      default (none)
###
###  Name of the file where the filter should write its pid before beginning
###  normal operations.
#
PidFile               /var/run/opendkim/opendkim.pid


# Always oversign From (sign using actual From and a null From to prevent
# malicious signatures header fields (From and/or others) between the signer
# and the verifier.  From is oversigned by default in the Debian pacakge
# because it is often the identity key used by reputation systems and thus
# somewhat security sensitive.
OversignHeaders         From

##  ResolverConfiguration filename
##      default (none)
##
##  Specifies a configuration file to be passed to the Unbound library that
##  performs DNS queries applying the DNSSEC protocol.  See the Unbound
##  documentation at http://unbound.net for the expected content of this file.
##  The results of using this and the TrustAnchorFile setting at the same
##  time are undefined.
##  In Debian, /etc/unbound/unbound.conf is shipped as part of the Suggested
##  unbound package

# ResolverConfiguration     /etc/unbound/unbound.conf

##  TrustAnchorFile filename
##      default (none)
##
## Specifies a file from which trust anchor data should be read when doing
## DNS queries and applying the DNSSEC protocol.  See the Unbound documentation
## at http://unbound.net for the expected format of this file.

TrustAnchorFile       /usr/share/dns/root.key

##  Userid userid
###      default (none)
###
###  Change to user "userid" before starting normal operation?  May include
###  a group ID as well, separated from the userid by a colon.
#
UserID                opendkim

```

### /etc/default/opendkim

```ini
# Command-line options specified here will override the contents of
# /etc/opendkim.conf. See opendkim(8) for a complete list of options.
#DAEMON_OPTS=""
# Change to /var/spool/postfix/var/run/opendkim to use a Unix socket with
# postfix in a chroot:
#RUNDIR=/var/spool/postfix/var/run/opendkim
RUNDIR=/var/run/opendkim
#
# Uncomment to specify an alternate socket
# Note that setting this will override any Socket value in opendkim.conf
# default:
#SOCKET=local:$RUNDIR/opendkim.sock
# listen on all interfaces on port 54321:
SOCKET=inet:8892@localhost
# listen on loopback on port 12345:
#SOCKET=inet:12345@localhost
# listen on 192.0.2.1 on port 12345:
#SOCKET=inet:12345@192.0.2.1
USER=opendkim
GROUP=opendkim
PIDFILE=$RUNDIR/$NAME.pid
EXTRAAFTER=

```

### SPF, DKIM, DMARC， RDNS配置
- DKIM配置  https://help.ubuntu.com/community/Postfix/DKIM  命令生成的内容里有一些多段双引号，删除双引号把内容连起来放入dns配置就可以了
- SPF配置
- DMARC配置 https://tools.socketlabs.com/dmarc/generator
- RDNS配置

### 测试
- SPF 测试https://mxtoolbox.com/spf.aspx
- DKMI 校验检测 https://dkimcore.org/tools/
- SPF、DMARC、DKMI测试 https://tools.wordtothewise.com/authentication
- 发送email测试打分 https://www.mail-tester.com/ 
- 发送email测试打分 https://dkimvalidator.com/
- PTR测试 https://toolbox.googleapps.com/apps/dig/#PTR/


### 参考资料
- 几个概念比较详细 https://www.smartertools.com/blog/2019/04/09-understanding-spf-dkim-dmarc
- 深入浅出 SPF，DKIM， DMARC https://wiki.zimbra.com/wiki/Best_Practices_on_Email_Protection:_SPF,_DKIM_and_DMARC




