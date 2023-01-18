# microcloud
microk8s fronted by a vpn gateway and reverse proxy

# ubuntu vpn gateway with reverse proxy
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
net.ipv4.conf.default.accept_redirects = 0

```


# ubuntu microk8s
