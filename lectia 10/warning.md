# warning

First, I extracted the authentication log: `tar --no-wildcards -xOf warning.tar '[root]/var/log/auth.log'`. The log contained multiple suspicious SSH `Invalid user` attempts originating from the same IP address: `192.168.181.158`

I filtered the relevant entries and extracted the attempted usernames:
```bash
tar --no-wildcards -xOf warning.tar '[root]/var/log/auth.log' |
grep 'Invalid user .* from 192.168.181.158' |
sed -E 's/.*Invalid user ([^ ]+) from.*/\1/'
```

This produced:
```text
S
S
S
Logstash
Oracle
Centos
Admin
Root
Deploy
Sysadmin
Postgres
Redis
informix
Nagios
Couchdb
itadmin
Pi
Ldap
elastic
Operator
Ftp
Ec2user
xguest
cacti
Hadoop
Ansible
Nginx
Git
esuser
```

Taking the first character of each username reveals the hidden message: `SSSLOCARDSPRiNCiPLeOFExcHANGe`
