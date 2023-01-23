# **microcloud**
microk8s fronted by a vpn gateway and reverse proxy

# ubuntu vpn gateway with reverse proxy
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


# ubuntu microk8s

## firewalld
```
sudo apt-get install firewalld
```
## cockpit
```
sudo apt-get install cockpit
```
## [microk8s](https://microk8s.io/docs/getting-started)
#### 1. install
```
sudo snap install microk8s --classic --channel=1.26
```
#### 2. join the group
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```
#### 3. re-enter the session
```
su - $USER
```
#### 4. calico vxlan overlay
```
sudo firewall-cmd --zone=trusted --add-interface=vxlan.calico --permanent
```
#### 5. calico pod networks
```
sudo firewall-cmd --zone=trusted --add-source=10.0.0.0/8  --permanent 
```
#### 6. microk8s servces for firewalld
```
sudo cp services/*.xml  /usr/lib/firewalld/services/ 
```
