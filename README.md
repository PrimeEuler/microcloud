# **microcloud**
A runbook to deploy a 3 node HA microk8s cluster (mk8s1-3) and a 2 node HA VPN gateway and reverse proxy (gw1-2) for access.
* The gateway nodes require 2 NIC's (outside & inside the cluster network).

```mermaid
%%{init: {'theme':'dark'}}%%
graph TD;
    A(vrrp);
    B(gw-1);
    C(gw-2);
    D(mk8s-1);
    E(mk8s-2);
    F(mk8s-3);
    A-->B & C;
    B & C-->D & E & F;
```

| ID  | TASK | DESCRIPTION | 
| --- | ---- | ----------- |
| [1](#cockpit) | Install cockpit on all nodes | Cockpit is a web-based graphical interface for servers | 
| [2](#netplan) | Configure network on all nodes | Gateway nodes get (inside) and (outside) networks. Microk8s nodes get (inside) network.
| [3](#firewalld) | Install firewalld on all nodes | Firewalld provides a dynamically managed firewall with support for network/firewall zones that define the trust level of network connections or interfaces | 
| [4](#libreswan) | Install libreswan on gateway nodes | Libreswan is a free software implementation of the most widely supported and standardized VPN protocol using "IPsec" and the Internet Key Exchange ("IKE") | 
| [5](#frrouting) | Install frrouting on gateway nodes | FRRouting (FRR) is a free and open source Internet routing protocol suite for Linux and Unix platforms. It implements BGP, OSPF, RIP, IS-IS, PIM, LDP, BFD, Babel, PBR, OpenFabric and VRRP, with alpha support for EIGRP and NHRP |
| [6](#haproxy) | Install haproxy on gateway nodes | HAProxy is a free, very fast and reliable reverse-proxy offering high availability, load balancing, and proxying for TCP and HTTP-based applications |
| [7](#microk8s) | Install microk8s on microk8s nodes | Microk8s is zero-ops, pure-upstream Kubernetes, from developer workstations to production. |


## [cockpit](https://cockpit-project.org/)
#### Start with the gateway to build access to the rest of the cluster. Onec the gateway is complete, the rest of hosts can be managed from cockpit. https://192.168.1.11:9090
```shell
sudo apt-get install cockpit

sudo systemctl start cockpit
```




## [netplan](https://netplan.io/)
```shell

# Gateway nodes configure (inside) & (outside) networks
sudo nano /etc/netplan/*.yaml
network:
  ethernets:
    eth0:
    # outside
      addresses:
      - 192.168.1.11/24
      routes:
      - to: default
        via: 192.168.1.1
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
        search:
        - .local
    eth1:
    # inside
      addresses:
      - 192.168.3.1/24
      # gw-1 192.168.3.2
      # gw-2 192.168.3.3
      # VRRP 192.168.3.1 (FRRouting)
  version: 2
  renderer: NetworkManager
  # cockpit uses NetworkManager
  
sudo netplan try
[enter]

# Microk8s nodes configure (inside) network
sudo nano /etc/netplan/*.yaml

network:
  ethernets:
    eth0:
    # inside 
      addresses:
      - 192.168.3.11/24
      routes:
      - to: default
        via: 192.168.3.1
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
        search:
        - .local
  version: 2
  renderer: NetworkManager
  # cockpit uses NetworkManager
  
sudo netplan try
[enter]



# All nodes hosts entries
sudo nano /etc/hosts

# gws zone = public
192.168.1.11 gw-1
192.168.1.12 gw-2

# gws zone = trusted 
192.168.3.1 = vrrp

# mk8s zone = public
192.168.3.11 mk8s-1
192.168.3.12 mk8s-2
192.168.3.13 mk8s-3


# All nodes NTP

sudo nano /etc/systemd/timesyncd.conf
[Time]
NTP=( Your Time Server )


```
## [firewalld](https://firewalld.org/)
```shell
sudo apt-get install firewalld

# enable logging
sudo firewall-cmd --set-log-denied=all

# enable cockpit
sudo firewall-cmd --add-service cockpit --permanent
sudo firewall-cmd --reload


# Enable IP Rorwarding on gateway nodes only
sudo nano /etc/sysctl.conf

net.ipv4.ip_forward=1

# Attackers could use bogus ICMP redirect messages to maliciously alter the system routing tables
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

sysctl -p

# Enable IP masquerade on gateway nodes only
sudo firewall-cmd --add-masquerade --permanent

# Accept all traffic on mk8s default gateway interface
firewall-cmd --permanent --zone=trusted --change-interface=(inside)

# apply
sudo firewall-cmd --reload
```
## [libreswan](https://libreswan.org/)
```shell
# ubuntu raspi extras not included in image
# sudo apt-get install linux-modules-extra-5.15.0-1017-raspi

sudo apt-get install libreswan

# ipsec tunnel to remote microcloud gateway
sudo cp ipsec.d/ocigw.cconf  /etc/ipsec.d/

```
## [frrouting](https://frrouting.org/)
```shell
sudo apt-get install frr

# Enable BGP
sudo nano /etc/frr/daemons

bgpd=yes

# bgp peering through ipsec tunnel to remote microcloud gateway
sudo cp frr/frr.conf  /etc/frr/

sudo systemctl restart frr
```
### [haproxy](https://www.haproxy.org/)
```shell
sudo apt-get install haproxy

# edit and load config for 80,443 with proxy protocol and 16443 for API access
sudo cp haproxy/haproxy.cfg  /etc/haproxy

sudo systemctl enable haproxy
sudo systemctl restart haproxy

```
### [microk8s](https://microk8s.io/docs/getting-started)
```shell
sudo snap install microk8s --classic --channel=1.26

# join the microk8s group
sudo usermod -a -G microk8s $USER

# take ownership of the config files
sudo chown -f -R $USER ~/.kube

# re-enter the session
su - $USER

# microk8s servces for firewalld
sudo cp services/*.xml  /usr/lib/firewalld/services/ 

sudo firewall-cmd --zone=public --permanent --add-service=http 
sudo firewall-cmd --zone=public --permanent --add-service=https 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-apiserver 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-calico 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-cluster-agent 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-dqlite 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-etcd 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-kube-controller 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-kube-scheduler 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-kubelet-r 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-kubelet-rw 
sudo firewall-cmd --zone=public --permanent --add-service=mk8s-observability 
sudo firewall-cmd --reload



# calico vxlan firewalld rules
sudo firewall-cmd --zone=trusted --add-interface=vxlan.calico --permanent

# calico pod networks
sudo firewall-cmd --zone=trusted --add-source=10.0.0.0/8  --permanent 
sudo firewall-cmd --reload

# Add VRRP address to certificates for external access
sudo nano /var/snap/microk8s/current/certs/csr.conf.template
# MOREIP
IP.9 = < VRRP IP >

# Microk8s should auto refresh the certs. verify cert conf file
sudo cat sudo nano /var/snap/microk8s/current/certs/csr.conf


# Add nodes to the cluster after all certs have been updated with VRRP IP
microk8s.add-node

# after cluster is formed enable dns, metric server, ingress and observability

microk8s.enable dns 
microk8s.enable ingrss
microk8s.enable metrics-server
microk8s.enable observability 

# add proxy protocol on ingress loadbalancer config
kubectl edit configmap -n ingress nginx-load-balancer-microk8s-conf

data:
  use-proxy-protocol: "true"

# https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
# install kubectl on gateway nodes to remote manage microk8s

# Download the Google Cloud public signing key
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add the Kubernetes apt repository:
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list\

# Update apt package index with the new repository and install kubectl:
sudo apt-get update
sudo apt-get install -y kubectl


# copy config to $HOME/.kube/config
microk8s.config

# edit cluster VIP adress for API 16443
sudo nano $HOME/.kube/config

server: https://192.168.1.11:16443

# add ingress rule for grafana observability access
kubectl apply -f microk8s/grafana-ingress.yaml




```

