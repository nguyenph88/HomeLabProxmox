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

# Combo: Cloudfare DDNS - Nginx - Cloudflare
1. Register for a cloudflare account -> get an API key
2. Get a new domain, point DNS to cloudflare 
3. Install cloudflare DDNS LXC as usual (set *.domain.com and domain.com) > do all the basic setup as needed
4. Install nginx LXC as usual > do all the basic setup as needed > this will fectch the local ip and update to cloudflare (just optional)

** Routing the network: 

Below i will route traffic from pve.domain.com to my 10.0.0.222:8006 which is my proxmox datacenter.

1. Open the gateway and open forwarding for 80 and 443 for the nginx local IP:port. (i.e: to 10.0.0.113:81)

HOW TO DEAL WITH XFINITY COMCAST PORT FORWARDING
> If you are using comcast xfinity like me they made it a pain in the butt. We cannot change that on the website but instead we need to use the xinifty app. Now on the freaking app, they don't list all the LXC device, neither can we enter the local ip directly.
> If you ever created a lxc container before (nginxproxymanager) then the name is taken up and will be cached, we cannot select that because it's associated with the wrong number.
> The way to deal with that is to add allthe "Proxmox" Devices in the Xfinity app (if you are running multiple lxc),then try to get the right one and delete the rest.
> <img width="820" height="557" alt="IMG_2071" src="https://github.com/user-attachments/assets/fb97ae4d-de72-4fbe-af85-fae30bc542fd" />



2. Open nginx > SSL Ceritificate > add a wildcard cert for the domain *.domain.com > Use DNS challenge = Cloudflare and use the API key from CF
3. Go to Hosts > Proxy Host > Add a proxy host > This is the subdomain you want the traffic to point to. In my case I want pve.domain.com to reverse to my proxmox at 10.0.0.222:8006

<img width="373" height="460" alt="Screenshot 2025-07-24 151029" src="https://github.com/user-attachments/assets/2d9a354e-7cf1-4432-a521-e9fc34717903" /> <img width="387" height="465" alt="Screenshot 2025-07-24 150749" src="https://github.com/user-attachments/assets/6d47ce6f-9bdb-45e4-a8af-c6162ccb0663" />

4. Go to cloudflare website, set the A record for the domain for pve (also set * and www if you want)
<img width="2019" height="115" alt="Screenshot 2025-07-24 152628" src="https://github.com/user-attachments/assets/35a0b98c-8a5b-4b05-bf3a-d69797855dfd" />

5. At this point this pve.domain.com should be accessible from outside of the network
6. Apply the same things for Plex, jellyfin or game server
   
# Cloudflare DDNS
To update the configuration edit `/etc/systemd/system/cloudflare-ddns.service`. After edit please restart with `systemctl restart cloudflare-ddns`

# Nginx
- If you cannot signin to proxmox with the 401 wrong ticket, needs to enabled SSL / HTTP2 support and force SSL for the domain

# Misc & Notes:
- Found a lot of cool scripts here: https://github.com/awesome-selfhosted/awesome-selfhosted?tab=readme-ov-file
- Use this website to check if the port 80 and 443 are open on the ip https://canyouseeme.org/
- Set SSL/TLS on Cloudflare to Full, otherwise streaming cannot work
