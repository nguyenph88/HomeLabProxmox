# HomeLabProxmox

Not comprehensive, just for my notes

Setting up my home lab with the following stack:
- Proxmox
- TrueNAS
- Plex

## Installing Proxmox
Easy
## Installing TrueNAS
Easy

- If have a new HDD, need to format it first. Check under Disks to see the correct name:
```
wipefs -a /dev/sdc (right name)
gdisk /dev/sdc (change the name) (type o , n , w)
```

- Don't start it yet, need to mount the storage first. In Proxmox main shell use these:
```
qm set 100 -scsi1 /dev/disk/by-id/ata-TOSHIBA_DT01ACA200_X4GVW1RTS
qm set 100 -scsi2 /dev/disk/by-id/ata-ST500DM002-1BD142_W2AYZ0QV
```

Keys takeaway:
- If using NFS share, the uid and gid in the linux system SHOULD match the user and gid in TrueNAS. To do so, create a new group and user in TrueNAS and force user/group create using these command: 
```
groupadd -g 3002 fsuser
useradd fsuser-u 3002 -g 3002
usermod — gid fsuser USERNAME_WE_WANT_TO_USE_LOCALLY
```
or just simply do
```
sudo useradd -m testuser
sudo passwd testuser
sudo usermod -aG sudo testb
```
- For Home<img width="3828" height="1655" alt="Screenshot 2025-07-22 125657" src="https://github.com/user-attachments/assets/7eaefdc9-ccb9-4b92-b157-9fd9a8c9a0ac" />
lab: just ignore the NFS share, it's for people who have lots of time and nothing to do so they can play around with those ids. We can just create a SMBuser and use the credentials to login.
- Try to set the network to a static IP like 10.0.0.2 so next time after reboot it won't change. Otherwise need to edit all the mount command.
<img width="3828" height="1655" alt="Screenshot 2025-07-22 125657" src="https://github.com/user-attachments/assets/45203dfb-ad8e-49db-99e7-a40827f382e6" />


# Installing Plex

>
>

- If create privileged LCX, need to make sure the "Nestling" is enabled.
