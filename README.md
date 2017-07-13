# SPOS
SPOS cheatsheet
```
Oprava   
sed -i.bak 's/^\#//g' /etc/resolv.conf #hostnames   
netstat -tupln #bezici procesy   
#mkfs.ext4 /dev/xvdb #naformatovani disku  

Zjisteni errorů   
journalctl -xe


LVM
apt install lvm2
pvcreate /dev/xvd
vgcreate data /dev/xvd
lvcreate -L 30G -n spos data

Raid
apt install mdadm
#udelat na diskach partisny
fdisk /dev/xdvb # n -> enter enter enter -> w # pro vsechny
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 missing
mkfs.ext4 /dev/md0

Image
dd if=/dev/zero of=/tmp/image bs=1M count=1024 #misto /dev/zero disk co chci zkopirovat
losetup -f
losetup -f /tmp/image
mkfs.ext4 /dev/loop0
mount  /dev/loop0 /mnt/nekde

#ip addr add 10.0.0.1/24 dev eth0

NFS
apt-get install nfs-server
#v /etc/exports
/tmp/kkt        10.0.0.1/24(rw) 10.0.0.2(ro) # dalsi parametry no_subtree_check,all_squash,anonuid=6000,anongid=6000
exportfs -a
#exportfs -r
exportfs -u # vypsat
#addgroup --gid 6000 --uid 6000 nfs #mapovani

Samba
useradd spos
smbpasswd -a spos
# na konec /etc/samba/smb.conf:
[share3]
        path = /srv/share3
        valid users = jindra
#        browsable = yes
#        writable = yes
#        guest ok = yes
#        create mask = 0600
#        directory mask = 0700
service smbd restart
#mount -t cifs //localhost/share1 /mnt/share -o username=jindra
#do fstabu -> //localhost/share-spos  /mnt/kkt2       cifs    username=jindra,password=jindra 0       0


Apache
apt-get install apache2
#v  /etc/apache2/ports.conf  zmwnit port 80 na napr localhost:9000
#v /etc/apache2/sites-available/000-default.confzmenit port a pripadne slozku (!! at zustane v /var/www/*)
#ve slozce /var/www vytworit index.hmtl
a2ensite 0*
service apache2 reload

Nginx
apt-get install nginx
# v /etc/nginx/sites-available/default upravit listen (adresa a port) a root kam to bude sahat


Pound
apt install pound
#!! v /etc/default/pound nastavit startup=1
#konfigurační soubor je: /etc/pound/pound.cfg

DNS
apt-get install bind9 dnsutils
#Nastavíme /etc/resolv.conf: nameserver 127.0.0.1
# v /etc/bind/named.conf.local nastavime
# normalni
zone "spos-test.spos"{
        type master;
        file "/etc/bind/db.spos1.spos";
};
#reverzni
zone "255.168.192.in-addr.arpa" IN {
        type master;
        file "/etc/bind/db.rev.spos";
};
#vytvorime soubory
###########################/etc/bind/db.spos1.spos

$TTL    604800
@       IN      SOA     spos-test.spos. admin.spos-test.spos. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      localhost.
@       IN      A       10.0.0.1

debian2 IN      A       10.0.0.1
www     IN      CNAME   debian2
mail    IN      A       10.0.0.1
@       IN      MX      10      mail
@       IN      MX      20      mail.lubos.spos.
;pokud chceme mit MX zaznamy pro "mail.test.spos"
mail       IN      MX      20      mail

##########################/etc/bind/db.rev.spos
$ORIGIN 255.168.192.in-addr.arpa.
$TTL    604800
@       IN      SOA     ns1.spos-test.spos. admin.spos-test.spos. (
                             45         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

        IN      NS      ns1.spos-test.spos.

55      IN      PTR     avrba.spos-test.spos.

##########################

#otocime bind
service bind9 restart
#overeni
host spos-name.spos
dig -x 192.168.255.55
```























