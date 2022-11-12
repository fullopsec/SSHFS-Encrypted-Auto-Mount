# sshfs
SSHFS: Automatically mounting remote encrypted partition with no password

on client and server:
sudo apt install sshfs 
sshfs -V

on client:

create the mount point

>sudo mkdir /media/databank  

>sudo chown daniel /media/databank 

set rights for the mount point

>chmod 700 /media/databank/

try to mount (the IP is your server IP)

>sshfs 192.168.1.35:/media/databank /media/databank

test that it works

>cd /media/databank/

>touch testtest

generate an Ed25519 key 
it was developed without any known government involvement. contrary to ECDSA
Stay well away from DSA (“ssh-dss”) keys as they are backdoored by the ₦⒮ⓐ

>ssh-keygen -t ed25519 -a 100

Copy your key to the server

>ssh-copy-id -i /home/daniel/.ssh/id_ed25519.pub daniel@192.168.1.35

try to connect via your key

>ssh daniel@192.168.1.35

edit your fstab on the client

>sudo nano /etc/fstab

of course change the IP and mount points so it correspond to yours
you can also remove or add arguments if you want

>daniel@192.168.1.35:/media/databank /media/databank fuse.sshfs x-systemd.automount,x-systemd.requires=network-online.target,_netdev,user,idmap=user,transform_symlinks,identityfile=/home/daniel/.ssh/id_ed25519,allow_other,default_permissions,uid=1000,gid=1000,exec 0 0

>cd /media

unmount everything

>sudo umount -a

remount everything: extremely important as it enable us to accept the key or it'll crash at reboot

>sudo mount -a               

set the rights again!

>chmod 700 /media/databank/

reboot the client

>sudo reboot now

when restarted verify everything works

>systemctl status media-databank.mount

>ls -l /media

verify auto reconnect if server fails by rebooting it and watch what happens with your client  when server rebooted

I use this techique on my fully encrypted servers and computers
I full encrypt all my installs(there is almost no performance cost and I never had a problem). It will greatly benefit your Opsec while being practical.

