# **microcloud**
A runbook to deploy a 3 node HA microk8s cluster (mk8s1-3) and a 2 node HA VPN gateway and reverse proxy (gw1-2) for access.
* The gateway nodes require 2 NIC's (inside,outside).

```mermaid
graph TD;
    A(vip);
    B(gw1);
    C(gw2);
    D(mk8s1);
    E(mk8s2);
    F(mk8s3);
    A-->B;
    A-->C;
    B-->D;
    B-->E;
    B-->F;
    C-->D;
    C-->E;
    C-->F;
```

| ID  | TASK | DESCRIPTION | 
| --- | ---- | ----------- |
| [1](#firewalld) | Install firewalld on ggateway nodes | Firewall to protect the gateway and cluster | 
| [2](#cockpit) | Install cockpit on gateway nodes | Web-based graphical interface for servers | 
| [3](#libreswan) | Install libreswan on gateway nodes | Libreswan is a free software implementation of the most widely supported and standardized VPN protocol using "IPsec" and the Internet Key Exchange ("IKE") | 
| [4](#frrouting) | Install frrouting on gateway nodes | FRRouting (FRR) is a free and open source Internet routing protocol suite for Linux and Unix platforms. It implements BGP, OSPF, RIP, IS-IS, PIM, LDP, BFD, Babel, PBR, OpenFabric and VRRP, with alpha support for EIGRP and NHRP |
| [5](#haproxy) | Install haproxy on gateway nodes | HAProxy is a free, very fast and reliable reverse-proxy offering high availability, load balancing, and proxying for TCP and HTTP-based applications |


### [firewalld](https://firewalld.org/)
```shell
sudo apt-get install firewalld
```
### [cockpit](https://cockpit-project.org/)
```shell
sudo apt-get install cockpit
```
### [libreswan](https://libreswan.org/)
```shell
# ubuntu raspi extras not included in image
# sudo apt-get install linux-modules-extra-5.15.0-1017-raspi

sudo apt-get install cockpit
```
### [frrouting](https://frrouting.org/)
```shell
sudo apt-get install frr
```
### [haproxy](https://www.haproxy.org/)
```shell
sudo apt-get install haproxy
```







### vpn gateway and reverse proxy
```
sudo apt-get install firewalld
sudo apt-get install cockpit
# ubuntu raspi extras
# sudo apt-get install linux-modules-extra-5.15.0-1017-raspi
sudo apt-get install libreswan
sudo apt-get install frr
sudo apt-get install haproxy

nano /etc/sysctl.conf

net.ipv4.ip_forward=1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0


nano /etc/frr/daemons 
# bgpd=no
bgpd=yes

```

### [microk8s](https://microk8s.io/docs/getting-started)
#### 1. install
```shell
sudo snap install microk8s --classic --channel=1.26
```
#### 2. join the microk8s group
```shell
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```
#### 3. re-enter the session
```shell
su - $USER
```
#### 2. calico vxlan overlay
```shell
sudo firewall-cmd --zone=trusted --add-interface=vxlan.calico --permanent
```
#### 3. calico pod networks
```shell
sudo firewall-cmd --zone=trusted --add-source=10.0.0.0/8  --permanent 
```
#### 4. microk8s servces for firewalld
```shell
sudo cp services/*.xml  /usr/lib/firewalld/services/ 
```
#### 5. log denied
```shell
sudo firewall-cmd --set-log-denied=all
```
### [cockpit](https://cockpit-project.org/)
```shell
sudo apt-get install cockpit
```
