## Bridging FiOS Gateway Router behind Ubiquiti Unifi Security Gateway or EdgeRouter Lite

Note that a Gateway router isn't necessary to accomplish the bridging; any old FiOS router will do. The Gateway router is actually overkill since it only serves the MOCA portion of the network (e.g., STBs), so if you can save some $$ by using a lower grade router, do it.

Make sure the existing FiOS router is working properly before starting.

### 1. Record information from the FiOS router
* My Network -> Network Connections -> Broadband Connection (Ethernet)
  * Record MAC Address and IP Address
* Disable Wireless Radios
* Enable remote administration through secondary HTTP Port (8443)
* Record Port forwarding

| Device            | Ports Forwarded |
|---                |---              |
|192.168.1.109:63145| UDP Any -> 63145|
|192.168.1.108:7547 | TCP Any -> 35000|
|192.168.1.107:7547 | TCP Any -> 35001|
|192.168.1.105:7547 | TCP Any -> 35002|
|192.168.1.106:7547 | TCP Any -> 35003|
|192.168.1.109:7547 | TCP Any -> 35004|
|127.0.0.1          | TCP Any -> 4567 |
|DiskStation: 5075  |
|DiskStation: 5076  |
|DiskStation:32400  |
|DiskStation:32475  |

* Record Static IP Assignments

| Device       | MAC               | IP Address |
|---           |---                | ---        |
| UAP Attic    | 04:18:d6:80:70:41 |  2         |
| UAP Kitchen  | 04:18:d6:80:70:3b |  3         |
| ~Ben Cam~    | c4:d6:55:36:ea:c5 |  5         |
| ~Noa Cam~    | c4:d6:55:35:f1:45 |  6         |
| Marantz      | 00:06:78:24:62:ab |  4         |
| DiskStation  | 00:11:32:19:9d:4f |  7         |
| Mac Mini     | c4:2c:03:04:b4:a1 |  8         |
| Mac Book Pro | 00:23:6c:92:b2:23 |  9         |


### 2. Prepare EdgeRouter or USG
A handful of resources to help...  
[Official Edge OS User Guide](https://dl.ubnt.com/guides/edgemax/EdgeOS_UG.pdf)  
[Configuring EdgeRouter by Larry Gadea](http://lg.io/2015/01/11/the-ubiquiti-edgerouter-configuring-this-extremely-lowcost-enterprisegrade-router-for-home-use.html)  
[EdgeRouter Lite Setup by Logan Marchione](https://loganmarchione.com/2016/04/ubiquiti-edgerouter-lite-setup/)  

#### Upgrade the USG Firmware
  Download latest firmware from [Ubiquiti](https://www.ubnt.com/download/unifi-switching-routing/usg)
  Copy to USG
  On local computer:
```
scp ~/Downloads/upgrade.tar ubnt@192.168.1.1:/home/ubnt/upgrade.tar
```
  On USG (via SSH):
```
sudo syswrapper.sh upgrade upgrade.tar
```
  Re-establish SSH, then confirm the version with `show version`

#### Change timezone
The list of time zones is under `usr/share/zoneinfo` if you need a different time zone.
```
configure
set system time-zone US/Eastern
commit
save

show system time-zone
```

#### Setup Ethernet interfaces
First configure the WAN on `eth0`. The USG requires little configuration here, but ERL may require more. See Logan Marchione for more info on ERL.
```
configure
show interfaces ethernet eth0
```
Next, configure the LAN interface on `eth1`. We're going to use 10.10.2.x IP range.
```
set interfaces ethernet eth1 address 10.10.2.1/24
delete interfaces ethernet eth1 address 192.168.1.1/24
set system static-host-mapping host-name setup.ubnt.com inet 10.10.2.1
delete system static-host-mapping host-name setup.ubnt.com inet 192.168.1.1
commit
save
```
Now, set up DHCP server on `eth1`
```
configure
set service dhcp-server shared-network-name LAN authoritative enable
set service dhcp-server shared-network-name LAN subnet 10.10.2.0/24 start 10.10.2.10 stop 10.10.2.199
set service dhcp-server shared-network-name LAN subnet 10.10.2.0/24 default-router 10.10.2.1
set service dhcp-server shared-network-name LAN subnet 10.10.2.0/24 dns-server 10.10.2.1
set service dhcp-server shared-network-name LAN subnet 10.10.2.0/24 lease 86400
```
USG has a default DHCP server set up, which needs to be deleted.
```
delete service dhcp-server shared-network-name LAN_192.168.1.0-24
commit
save
show service dhcp-server
```
 
Set up DNS servers (Google? OpenDNS? OpenNIC? See Logan Marchione for more)

Fix the default NAT to work with our IP subnet.
```
set service nat rule 5000 source address 10.10.2.1/24
commit
save
```

#### Set up SSH keys
On local machine:
```
scp ~/.ssh/id_rsa.pub ubnt@192.168.1.1:~/id_rsa.pub
```
Then, load the key and turn off password authentication.
```
configure
loadkey ubnt ~/id_rsa.pub
set service ssh disable-password-authentication
commit
save
```

#### Clone MAC Address and IP of FiOS router
```
configure
set interfaces ethernet eth0 mac c8:a7:0a:9f:f7:c3
set interfaces ethernet eth0 address xxx.xxx.xxx.xxx
commit
save
```

#### Set Static DHCP leases
```
configure
set service dhcp-server hostfile-update enable
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping FiOS ip-address 10.10.2.5
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping FiOS mac-address 6c:b0:ce:43:cb:8d 
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping UAPAttic ip-address 10.10.2.2
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping UAPAttic mac-address 04:18:d6:80:70:41
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping UAPKitchen ip-address 10.10.2.3
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping UAPKitchen mac-address 04:18:d6:80:70:3b
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping Marantz ip-address 10.10.2.4
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping Marantz mac-address 00:06:78:24:62:ab
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping DiskStation ip-address 10.10.2.7
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping DiskStation mac-address 00:11:32:19:9d:4f
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping MacMini ip-address 10.10.2.8
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping MacMini mac-address c4:2c:03:04:b4:a1
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping MacBookPro ip-address 10.10.2.9
set service dhcp-server shared-network-name LAN_10.10.2.0-24 subnet 10.10.2.0/24 static-mapping MacBookPro mac-address 00:23:6c:92:b2:23
commit
save
```

#### Set Port Forwarding
First, set some basic port-forwarding settings
```
configure
set port-forward auto-firewall enable
set port-forward hairpin-nat enable
set port-forward lan-interface eth1
set port-forward wan-interface eth0
commit
```

Next, set up the port-forwarding that FiOS requires. I'm forwarding these ports to the backend router. The ports and protocols are based on the information I recorded from the FiOS router in step 1 above. Your requirements may be different than mine.
```
configure
set port-forward rule 1 description 'FiOS 63145'
set port-forward rule 1 forward-to address 10.10.2.5
set port-forward rule 1 forward-to port 63145
set port-forward rule 1 original-port 63145
set port-forward rule 1 protocol udp
set port-forward rule 2 description "FiOS 35000"
set port-forward rule 2 forward-to address 10.10.2.5
set port-forward rule 2 forward-to port 35000
set port-forward rule 2 original-port 35000
set port-forward rule 2 protocol tcp
set port-forward rule 3 description "FiOS 35001"
set port-forward rule 3 forward-to address 10.10.2.5
set port-forward rule 3 forward-to port 35001
set port-forward rule 3 original-port 35001
set port-forward rule 3 protocol tcp
set port-forward rule 4 description "FiOS 35002"
set port-forward rule 4 forward-to address 10.10.2.5
set port-forward rule 4 forward-to port 35002
set port-forward rule 4 original-port 35002
set port-forward rule 4 protocol tcp
set port-forward rule 5 description "FiOS 35003"
set port-forward rule 5 forward-to address 10.10.2.5
set port-forward rule 5 forward-to port 35003
set port-forward rule 5 original-port 35003
set port-forward rule 5 protocol tcp
set port-forward rule 6 description "FiOS 35004"
set port-forward rule 6 forward-to address 10.10.2.5
set port-forward rule 6 forward-to port 35004
set port-forward rule 6 original-port 35004
set port-forward rule 6 protocol tcp
set port-forward rule 7 description "FiOS 4567"
set port-forward rule 7 forward-to address 10.10.2.5
set port-forward rule 7 forward-to port 4567
set port-forward rule 7 original-port 4567
set port-forward rule 7 protocol tcp
```

Now, set up any other port forwarding necessary.
```
set port-forward rule 10 description ""
```

### 3. Prepare back-end router
I'm using a Netgear R6250 for my back-end router. This is probably overkill, but I happened to have one sitting around.
* Disable Wireless radios
  "Advanced" tab -> "Advanced Setup" -> "Wireless Settings" -> uncheck "Enable Wireless Router Radio" for 2.4 GHz and 5 GHz radios.
* Set up LAN
  "Advanced" tab
  "LAN Setup"
    IP Address: XXX.XXX.XXX.1 to match up with Broadband IP from FiOS router
    Apply
    Address Reservation -> Add
    Add IP Broadband IP and Mac Address from FiOS Router
* Configure Port Forwarding rules
  Forward the FiOS DVR (63145), FiOS Admin (4567) and FiOS TV ports (35000-35004) to the FiOS Gateway Router

* Turn on Remote Management ("Advanced" tab -> "Advanced Setup" -> "Remote Management")
  Remote management for Netgear at https://10.10.2.5:8443

### 4. Install replacement router

### 5. Install back-end router

### 6. Configure FiOS Gateway Router

### 7. Install FiOS Gateway Router

