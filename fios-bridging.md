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

* Record Static IP Assignments

| Device       | MAC               | IP Address |
|---           |---                | ---        |
| UAP Attic    | 04:18:d6:80:70:41 |  2         |
| UAP Kitchen  | 04:18:d6:80:70:3b |  3         |
| Ben Cam      | c4:d6:55:36:ea:c5 |  5         |
| Noa Cam      | c4:d6:55:35:f1:45 |  6         |
| Marantz      | 00:06:78:24:62:ab |  8         |
| DiskStation  | 00:11:32:19:9d:4f |  10        |
| Mac Mini     | c4:2c:03:04:b4:a1 |  11        |
| Mac Book Pro | 00:23:6c:92:b2:23 |  12        |


### 2. Prepare EdgeRouter or USG
A handful of resources to help...  
[Official Edge OS User Guide](https://dl.ubnt.com/guides/edgemax/EdgeOS_UG.pdf)  
[Configuring EdgeRouter by Larry Land](http://lg.io/2015/01/11/the-ubiquiti-edgerouter-configuring-this-extremely-lowcost-enterprisegrade-router-for-home-use.html)  
[EdgeRouter Lite Setup by Logan Marchione](https://loganmarchione.com/2016/04/ubiquiti-edgerouter-lite-setup/)  

* Clone MAC Address and IP of FiOS router
* 


### 3. Prepare back-end router

### 4. Install replacement router

### 5. Install back-end router

### 6. Configure FiOS Gateway Router

### 7. Install FiOS Gateway Router

