# Create your own Web Proxy Server
```sh
yum update -y
sudo fallocate -l 1G /var/swap.img ; chmod 600 /var/swap.img
mkswap /var/swap.img ; swapon /var/swap.img
echo "/var/swap.img    none    swap    sw    0    0" >> /etc/fstab
yum install -y httpd mod_ssl
nano /etc/httpd/conf.d/forward-proxy.conf
```
```sh
ProxyRequests On
ProxyVia On
ProxyTimeout 60

<Proxy *>
    Require ip ::1
    Require all granted
</Proxy>
```
```sh
service httpd start ; systemctl enable httpd.service
```
