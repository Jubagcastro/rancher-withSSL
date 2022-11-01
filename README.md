# Rancher on Docker-withSSL
Repo to document step-by-step rancher w/ cert
To follow this step-by-step you should have a DNS purchased and an account in Cloudflare

# Step 1 - Pre-req
Ubuntu 20.04

## Install
- qemu-guest-agent
- nfs-common
- ntp
- zsh
- p10k
- zsh-syntax-highlighting
- docker
 
# Step 2 - Certs
- Create cloudflare account
- Generate Cert via acme.sh
```bash
curl https://get.acme.sh | sh -s email=my@email.com
```
- Export Cloudflare's auth
```bash
> export CF_Account_ID="#####"
> export CF_Token="#####"
> export CF_Zone_ID="#####"
> .acme.sh/acme.sh --issue --dns dns_cf -d <your.domain.com> -d <www.your.domain.com>
> .acme.sh/acme.sh --cron 
> cp ~/.acme.sh/<YOUR DOMAIN>/fullchain.cer ~/.acme.sh/<YOUR DOMAIN>/fullchain.pem 
> cp ~/.acme.sh/<YOUR DOMAIN>/<YOUR DOMAIN>.key ~/.acme.sh/<YOUR DOMAIN>/key.pem 
```

# Step 3 - Rancher
Optional: Persist storage on a NFS
```bash
sudo mount -t nfs <YOUR NAS IP/NAME>:<YOUR EXPORT PATH>/rancher /opt/rancher
```
Don't forget to set it in your fstab to mount at startup
```bash
> sudo vim /etc/fstab
<YOUR NAS IP/NAME:<YOUR EXPORT PATH>/rancher               /opt/rancher      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```
Rancher start Docker single node
```bash
sudo docker run -e TZ=America/Sao_Paulo \
-d --restart=unless-stopped \
-v ~/.acme.sh/<DOMAIN>/fullchain.pem:/etc/rancher/ssl/cert.pem \
-v ~/.acme.sh/<YOUR DOMAIN>/key.pem:/etc/rancher/ssl/key.pem \
-v /opt/rancher:/var/lib/rancher \
-p 80:80 -p 443:443 --privileged \
rancher/rancher:latest \
--no-cacerts
```
