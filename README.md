#!/bin/bash
fdisk /dev/vdb <<EOF
n
p
  
  
+512M
n
p
  
  
+1G
n
p
  

+1G
w
EOF
vgcreate systemvg /dev/vdb2
lvcreate -n vo -L 300 systemvg
mkfs.ext3 /dev/systemvg/vo
sed -i '1,1a /dev/systemvg/vo /vo ext3 defaults 0 0' /etc/fstab
mkdir /vo
mount -a
hostnamectl set-hostname server0.example.com
nmcli connection modify "System eth0"  ipv4.method manual ipv4.addresses "172.25.0.11/24 172.25.0.254" ipv4.dns 172.25.254.254 connection.autoconnect yes
nmcli connection up "System eth0"
groupadd adminuser
useradd -G adminuser natasha
useradd -G adminuser harry
useradd -s /sbin/nologin  sarah
echo flectrag | passwd --stdin natasha
echo flectrag | passwd --stdin harry
echo flectrag | passwd --stdin sarah
cp /etc/fstab /var/tmp/fstab
setfacl -m u:natasha:rw /var/tmp/fstab
setfacl -m u:harry:- /var/tmp/fstab
sed -i '15,15a 23 14 * * * /bin/echo hiya' /etc/crontab
crontab -u natasha /etc/crontab
mkdir /home/admins
chown :adminuser /home/admins
chmod 2770 /home/admins
wget http://classroom.example.com/content/rhel7.0/x86_64/errata/Packages/kernel-3.10.0-123.1.2.el7.x86_64.rpm
rpm -ivh kernel-3.10.0-123.1.2.el7.x86_64.rpm
yum -y install sssd
echo '[domain/default]

autofs_provider = ldap
cache_credentials = True
krb5_realm = #
ldap_search_base = dc=example,dc=com
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldap://classroom.example.com/
ldap_id_use_start_tls = True
ldap_tls_cacertdir = /etc/openldap/cacerts
[sssd]
services = nss, pam, autofs
config_file_version = 2

domains = default
[nss]

[pam]

[sudo]

[autofs]

[ssh]

[pac]' > /etc/sssd/sssd.conf
mkdir /etc/openldap/cacerts
chmod 0600 /etc/sssd/sssd.conf
cd /etc/openldap/cacerts
wget  http://classroom.example.com/pub/example-ca.crt
cd ~
echo 'IPADOMAINJOINED=no
USEMKHOMEDIR=no
USEPAMACCESS=no
CACHECREDENTIALS=yes
USESSSDAUTH=no
USESHADOW=yes
USEWINBIND=no
USESSSD=yes
PASSWDALGORITHM=sha512
FORCELEGACY=no
USEFPRINTD=no
FORCESMARTCARD=no
USELDAPAUTH=yes
USEPASSWDQC=no
IPAV2NONTP=no
WINBINDKRB5=no
USELDAP=yes
USEECRYPTFS=no
USEIPAV2=no
USEWINBINDAUTH=no
USESMARTCARD=no
USELOCAUTHORIZE=yes
USENIS=no
USEKERBEROS=no
USESYSNETAUTH=no
USEDB=no
USEPWQUALITY=yes
USEHESIOD=no' > /etc/sysconfig/authconfig
systemctl restart sssd
systemctl enable sssd
yum -y install autofs
mkdir /home/guests
echo '/home/guests /etc/auto.guests' >> /etc/auto.master
echo '* -rw classroom.example.com:/home/guests/&' > /etc/auto.guests
systemctl restart autofs
systemctl enable autofs
yum -y install chrony
sed -i '4,6d' /etc/chrony.conf
echo server classroom.example.com iburst >> /etc/chrony.conf
useradd -u 3456 alex
echo flectrag | passwd --stdin alex
mkdir /root/findfiles
find / -user student -type f -exec cp -p {} /root/findfiles/ \;
grep seismic /usr/share/dict/words > /root/wordlist
vgcreate -s 16M datastore /dev/vdb3
lvcreate -l 50 -n database datastore
mkfs.ext3 /dev/datastore/database
mkdir /mnt/database
echo /dev/datastore/database /mnt/database ext3 defaults 0 0 >> /etc/fstab
mount -a
tar -jcPf /root/backup.tar.bz2 /usr/local/
mkswap /dev/vdb1
echo /dev/vdb1 swap swap defaults 0 0 >> /etc/fstab
swapon -a
reboot
