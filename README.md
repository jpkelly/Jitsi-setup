# Jitsi-setup AWS EC2 Ubuntu Server 18.04 
Steps to set up Jitsi from Jitsi Self Hosting Guide

## Install Jitsi Meet

```
sudo -i
```

```
apt update
```

```
apt install apt-transport-https
```

```
sudo hostnamectl set-hostname <FQDN>
```
  
Edit `/etc/hosts`
```
127.0.0.1 localhost
<PUBLICIP> <FQDN> <SUBDOMAIN>
```

```
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null

# update all package sources
sudo apt update
```

```
sudo apt install jitsi-meet
```

```
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

Edit `/etc/jitsi/videobridge/sip-communicator.properties`
```
org.ice4j.ice.harvest.DISABLE_AWS_HARVESTER=false
```

Edit `/etc/systemd/system.conf`
```
DefaultLimitNOFILE=65000
DefaultLimitNPROC=65000
DefaultTasksMax=65000
```

```
reboot
```

## Failover to from 10000 to 443 is broken so ...
