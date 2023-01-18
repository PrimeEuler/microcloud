# microcloud
microk8s fronted by a VPN gateway and proxy

# Ubuntu VPN Gateway with Reverse Proxy
```
sudo apt-get install firewalld
sudo apt-get install libreswan
sudo apt-get install frr
sudo apt-get install haproxy

nano /etc/sysctl.conf

net.ipv4.ip_forward=1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.wlan0.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.wlan0.accept_redirects = 0
```


# Ubuntu microk8s
