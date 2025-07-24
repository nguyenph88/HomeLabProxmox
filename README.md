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
- For Homelab: just ignore the NFS share, it's for people who have lots of time and nothing to do so they can play around with those ids. We can just create a SMBuser and use the credentials to login. Create smbuser and give it all permission.
<img width="3822" height="1846" alt="Screenshot 2025-07-22 131105" src="https://github.com/user-attachments/assets/0d8b2ff6-5894-4bc8-a869-c780a348d1a0" />


- Try to set the network to a static IP like 10.0.0.2 so next time after reboot it won't change. Otherwise need to edit all the mount command.
<img width="3828" height="1655" alt="Screenshot 2025-07-22 125657" src="https://github.com/user-attachments/assets/45203dfb-ad8e-49db-99e7-a40827f382e6" />

# Sharing the dataset from TrueNAS with other LCX
Here is the pain, there are 2 types of LCX: Privileged  and un Privileged. Most of the applications are created as unprivileged. If we do that, there is no direct freaking way to mount the NAS dataset. There are 2 options:
1. Create a Privileged LCX
2. Need to mount the dataset to the Proxmox host and passthrough to LCX

How to do 2: (only run command in host shell)
- Run
```
apt install nfs-common
apt install cifs-utils
mkdir /PlexMedia
nano /etc/fstab
```
- Add this to fstab
```
//10.0.0.2/PlexMedia /PlexMedia cifs username=root,password=123456,uid=100000,gid=100000 0 0

#If using NFS then use this: (without password)
#10.0.0.2:/mnt/HDD500G1/PlexMedia /PlexMedia nfs defaults,_netdev 0 0
```
> Why uid and gid = 100000? bcause we are running from the unprivileged LCX, the proxmox acts as a middleman but it does not translate the root (0, 0) as root, it instead automatically change to 100k to make the host secured. 
>So we gotta force it, otherwise we can only read, and cannot write.
> It also means, if we have a user in LCX with UID 1000, and we want to pass it through to TrueNAS, the uid and gid should be 101000.
> Thanks me later!
> 
- Mount that to the lcx
```
pct set 101 -mp0 /PlexMedia,mp=/PlexMedia

#101 is the LCX container id
#first /PlexMedia is dir from host
#second /PlexMedia is dir on the LCX
```

- I learned a lot from these 2 videos
```
https://www.youtube.com/watch?v=Hu1fY0-FvVE
https://www.youtube.com/watch?v=CFhlg6qbi5M
```

# Installing Plex
- Use the community helper script to install the LCX https://community-scripts.github.io/
>
>

- If create privileged LCX, need to make sure the "Nestling" is enabled.

# Misc & Notes:
- Found a lot of cool scripts here: https://github.com/awesome-selfhosted/awesome-selfhosted?tab=readme-ov-file
