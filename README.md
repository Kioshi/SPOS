# SPOS
SPOS cheatsheet

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





