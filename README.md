# Jitsi-setup AWS EC2 Ubuntu Server 18.04 
Steps to set up Jitsi from Jitsi Self Hosting Guide

## Prepare network environment

```
sudo -i
```

```
hostnamectl set-hostname <FQDN>
```

```
apt update
```

```
apt install apt-transport-https
```


#### Edit `/etc/hosts`
```
127.0.0.1 localhost
<PUBLICIP> <FQDN> <SUBDOMAIN>
```

## Install correct version of Prosody first
https://community.jitsi.org/t/how-to-how-do-i-update-prosody/72205


## Install Jitsi Meet
```
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null
```

```
apt update
```

```
sudo apt install jitsi-meet
```

```
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

#### Edit `/etc/jitsi/videobridge/sip-communicator.properties`
```
org.ice4j.ice.harvest.DISABLE_AWS_HARVESTER=false
```

#### Edit `/etc/systemd/system.conf`
```
DefaultLimitNOFILE=65000
DefaultLimitNPROC=65000
DefaultTasksMax=65000
```

```
reboot
```

## Failover to from 10000 to 443 is broken so ...
From https://community.jitsi.org/t/coturn-chronicles/73099

#### Edit `/etc/turnserver.conf` (added/changed lines at bottom)
```
# jitsi-meet coturn config. Do not modify this line
use-auth-secret
keep-address-family
static-auth-secret=<VERYSECRET>
realm=<FQDN>
# cert=/etc/coturn/certs/<FQDN>.fullchain.pem
# pkey=/etc/coturn/certs/<FQDN>.privkey.pem
no-multicast-peers
no-cli
no-loopback-peers
no-tcp-relay
no-tcp
listening-port=3478
tls-listening-port=5349
# external-ip=<PUBLICIP>
no-tlsv1
no-tlsv1_1
# https://ssl-config.mozilla.org/#server=haproxy&version=2.1&config=intermediate&openssl=1.1.0g&guideline=5.4
cipher-list=ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
# jitsi-meet coturn relay disable config. Do not modify this line
denied-peer-ip=0.0.0.0-0.255.255.255
denied-peer-ip=10.0.0.0-10.255.255.255
denied-peer-ip=100.64.0.0-100.127.255.255
denied-peer-ip=127.0.0.0-127.255.255.255
denied-peer-ip=169.254.0.0-169.254.255.255
denied-peer-ip=127.0.0.0-127.255.255.255
denied-peer-ip=172.16.0.0-172.31.255.255
denied-peer-ip=192.0.0.0-192.0.0.255
denied-peer-ip=192.0.2.0-192.0.2.255
denied-peer-ip=192.88.99.0-192.88.99.255
denied-peer-ip=192.168.0.0-192.168.255.255
denied-peer-ip=198.18.0.0-198.19.255.255
denied-peer-ip=198.51.100.0-198.51.100.255
denied-peer-ip=203.0.113.0-203.0.113.255
denied-peer-ip=240.0.0.0-255.255.255.255
syslog

cert=/etc/letsencrypt/live/<FQDN>/fullchain.pem
pkey=/etc/letsencrypt/live/<FQDN>/privkey.pem
external-ip=<PUBLICIP>/<PRIVATEIP>
listening-ip=127.0.0.1
allowed-peer-ip=<PRIVATEIP>
no-udp
```

```
reboot
```
## Enable JWT

Install lua components
```
cd &&
apt-get update -y &&
apt-get install gcc -y &&
apt-get install unzip -y &&
apt-get install lua5.2 -y &&
apt-get install liblua5.2 -y &&
apt-get install luarocks -y &&
luarocks install basexx &&
apt-get install libssl1.0-dev -y &&
luarocks install luacrypto &&
mkdir src &&
cd src &&
luarocks download lua-cjson &&
luarocks unpack lua-cjson-2.1.0.6-1.src.rock &&
cd lua-cjson-2.1.0.6-1/lua-cjson &&
sed -i 's/lua_objlen/lua_rawlen/g' lua_cjson.c &&
sed -i 's|$(PREFIX)/include|/usr/include/lua5.2|g' Makefile &&
luarocks make &&
luarocks install luajwtjitsi &&
cd
```

#### Edit `/etc/prosody/conf.avail/<FQDN>.cfg.lua` 
VirtualHost <FQDN>
```
        authentication = "token"
        app_id="<APP_ID>"
        app_secret="<APP_SECRET>"
```
Create guest VirtualHost (without requiring auth)
```
  VirtualHost "guest.<FQDN>"
    authentication = "anonymous"
    modules_enabled = {
            "bosh";
            "pubsub";
            "ping"; -- Enable mod_ping
            "speakerstats";
            "turncredentials";
            "conference_duration";
            "muc_lobby_rooms";
        }
    c2s_require_encryption = false
```

#### Edit `/etc/jitsi/meet/<FQDN>-config.js` to enable anonymousdomain
```
anonymousdomain: 'guest.<FQDN>',
```
  
#### Add to `/etc/jitsi/jicofo/sip-communicator.properties`
```
org.jitsi.jicofo.auth.URL=EXT_JWT:<FQDN>
```

#### Edit `/etc/prosody/prosody.cfg.lua`
```
admins = { }

component_ports = { 5347 }
component_interface = "0.0.0.0"
```

## Token Moderator Plugin
https://github.com/nvonahsen/jitsi-token-moderation-plugin
