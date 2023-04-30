
# SSHFS - Auto mount remote encrypted partition

This repository contains instructions for automatically mounting a remote encrypted partition with no password using SSHFS. This technique can be used on fully encrypted servers and computers to greatly benefit your operational security while being practical.

## Prerequisites
Before using SSHFS, make sure to have it installed on both the client and server. You can do so by running the following command on both machines:
```
sudo apt install sshfs 
sshfs -V
```

## Mounting the remote partition
On the client machine, create the mount point and set the rights for the mount point using the following commands:
```
sudo mkdir /media/databank
sudo chown daniel /media/databank
chmod 700 /media/databank/
```

Then, try to mount the remote partition using SSHFS. Replace the IP address with the server's IP address:
```
sshfs 192.168.1.35:/media/databank /media/databank
```

Verify that it works by navigating to the mount point and creating a test file:
```
cd /media/databank/
touch testtest
```

## Generating an Ed25519 key
To generate an Ed25519 key, run the following command on the client machine:
```
ssh-keygen -t ed25519 -a 100
```

Copy your key to the server using the following command, replacing the path to the key and the IP address with your own:
```
ssh-copy-id -i /home/daniel/.ssh/id_ed25519.pub daniel@192.168.1.35
```

Try to connect to the server using your key:
```
ssh daniel@192.168.1.35
```

## Editing fstab on the client
Edit your fstab on the client machine using the following command, replacing the IP address and mount points with your own:
```
sudo nano /etc/fstab
```

Add the following line to your fstab, again replacing the IP address and mount points:
```
daniel@192.168.1.35:/media/databank /media/databank fuse.sshfs x-systemd.automount,x-systemd.requires=network-online.target,_netdev,user,idmap=user,transform_symlinks,identityfile=/home/daniel/.ssh/id_ed25519,allow_other,default_permissions,uid=1000,gid=1000,exec 0 0
```

## Unmounting and remounting
Unmount everything using the following command:
```
sudo umount -a
```

Remount everything using the following command. This is extremely important as it enables us to accept the key or it'll crash at reboot:
```
sudo mount -a
```

Set the rights again using the following command:
```
chmod 700 /media/databank/
```

## Reboot and verification
Reboot the client machine using the following command:
```
sudo reboot now
```

After restarting, verify that everything works using the following commands:
```
systemctl status media-databank.mount
ls -l /media
```

To verify auto-reconnect if the server fails, reboot it and watch what happens with your client machine when the server is rebooted.
