
# SSHFS - Auto mount remote encrypted partition

This repository contains instructions for automatically mounting a remote encrypted partition with no password using SSHFS. This technique can also be used on fully encrypted servers and computers to greatly benefit your operational security while being practical.

## Video tutorial

https://www.youtube.com/watch?v=gEeHH7n07YE

## Prerequisites
Before using SSHFS, make sure to have it installed on both the client and server. You can do so by running the following command on both machines:
```
sudo apt install sshfs 
sshfs -V
```

## Mounting the remote partition
On the client machine, create the mount point and set the rights for the mount point using the following commands:

#### Format

```
sudo mkdir <MOUNT_POINT>
sudo chown <USERNAME> <MOUNT_POINT>
chmod 700 <MOUNT_POINT>
```

#### Example

```
sudo mkdir /media/databank
sudo chown daniel /media/databank
chmod 700 /media/databank/
```


Then, try to mount the remote partition using SSHFS. Replace the IP address with the server's IP address:

#### Format

```
sshfs <SERVER_IP>:<MOUNT_POINT> <MOUNT_POINT>
```

#### Example

```
sshfs 192.168.1.35:/media/databank /media/databank
```


Verify that it works by navigating to the mount point and creating a test file:
```
cd <MOUNT_POINT>
touch test
```

## Generating an Ed25519 key

Ed25519 was developed without any known government involvement. Contrary to ECDSA.
Stay well away from DSA (“ssh-dss”) keys as they are cryptographically backdoored by intelligence services.

To generate an Ed25519 key, run the following command on the client machine:
```
ssh-keygen -t ed25519 -a 100
```
Do not put any password, we want to mount it automatically at boot. I will show at the end how you can secure this key a bit more.

Copy your key to the server using the following command, replacing the path to the key and the IP address with your own:

#### Format

```
ssh-copy-id -i /home/<USER>/.ssh/id_ed25519.pub <USER>@<SERVER_IP>
```

#### Example

```
ssh-copy-id -i /home/daniel/.ssh/id_ed25519.pub daniel@192.168.1.35
```

Try to connect to the server using your key:
```
ssh <USER>@<SERVER_IP>
```

## Editing fstab on the client
Edit your fstab on the client machine using the following command, replacing the IP address and mount points with your own:
```
sudo nano /etc/fstab
```

Add the following line to your fstab, again replacing the IP address and mount points:

#### Format

```
<user>@<server_ip>:<MOUT_POINT> <MOUT_POINT> fuse.sshfs x-systemd.automount,x-systemd.requires=network-online.target,_netdev,user,idmap=user,transform_symlinks,identityfile=<SSH_KEY>,allow_other,default_permissions,uid=1000,gid=1000,exec 0 0
```

#### Example
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
chmod 700 <MOUT_POINT>
```

## Reboot and verification
Reboot the client machine using the following command:
```
sudo reboot now
```

After restarting, verify that everything works using the following commands:

#### Format
```
systemctl status media-<DISK_NAME>.mount
ls -l /media
```

#### Example
```
systemctl status media-databank.mount
ls -l /media
```

To verify auto-reconnect if the server fails, reboot it and watch what happens with your client machine when the server is rebooted.

## Securing the key

Using an SSH key with no password is convenient, but it also means that anyone with access to your private key can gain access to your system without needing a password. To secure your SSH key, you can:

#### Restrict file permissions:

You can restrict the permissions on your private key file to prevent unauthorized access. Only allow root to use the key.

```
sudo chown root:root id_ed25519
sudo chmod 000 id_ed25519
```

#### Limit SSH access:

You can limit SSH access to your server by using firewall rules or by configuring SSH to only allow connections from specific IP addresses or users. 

#### Restrict key access: 

You could allow this ssh key to only access the mountpoint.

To restrict the access of a particular SSH key to a specific folder, you can add a command option to the SSH public key in the authorized_keys file.

Keep in mind that the user will still be able to see the contents of the restricted directory and execute any commands that are allowed within that directory, so it's important to carefully consider the permissions and contents of the directory.
