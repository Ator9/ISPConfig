CentOS 7 - Multiserver - Admin
```sh
my_ssh_user=XXX
```
#1. Startup
```sh
echo "IP     adm.server.com" >> /etc/hosts
echo "IP     db.server.com" >> /etc/hosts
echo "IP     http.server.com" >> /etc/hosts
service network restart
```
```sh
yum update -y
yum install -y telnet nmap quota ntp epel-release git
yum install -y clamav clamav-update rkhunter
yes | cp /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime
sed -i -e 's/#PermitRootLogin yes/PermitRootLogin without-password/' /etc/ssh/sshd_config
service sshd restart
adduser $my_ssh_user ; passwd $my_ssh_user
```

#2. Swap, quota, fail2ban & firewall
```sh
sudo fallocate -l 1G /var/swap.img ; chmod 600 /var/swap.img
mkswap /var/swap.img ; swapon /var/swap.img
echo "/var/swap.img    none    swap    sw    0    0" >> /etc/fstab
sed -i '0,/defaults/s//defaults,usrquota,grpquota/' /etc/fstab
mount -o remount /
quotacheck -avugm ; quotaon -avug
yum install -y fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sed -i -e 's/bantime  = 600/bantime  = 3600/g' /etc/fail2ban/jail.local
systemctl enable fail2ban.service ; systemctl start fail2ban.service
systemctl stop firewalld.service ; systemctl disable firewalld.service
```

#3. Install MariaDB
```sh
yum install -y mariadb-server
service mariadb start ; systemctl enable mariadb.service
/usr/bin/mysql_secure_installation
```

#4. Install Apache & PHP
```sh
yum install -y httpd mod_ssl
service httpd start ; systemctl enable httpd.service
yum install -y php php-devel php-gd php-imap php-ldap php-mssql php-mysql php-odbc php-pear php-xml php-xmlrpc php-pecl-apc php-mbstring php-mcrypt php-snmp php-soap php-tidy curl curl-devel perl-libwww-perl ImageMagick libxml2 libxml2-devel php-cli httpd-devel unzip bzip2 perl-DBD-mysql php-fpm mod_fcgid
systemctl start php-fpm.service ; systemctl enable php-fpm.service
service httpd restart
```

#5. Install phpMyAdmin
```sh
yum install -y phpmyadmin
sed -i -e 's/Require ip ::1/Require ip ::1\nRequire all granted/' /etc/httpd/conf.d/phpMyAdmin.conf
service httpd restart
```

#6. MySQL GRANT Access to other servers
```sh
mysql -hlocalhost -uroot -pYOUR_PASSWORD
```
```sh
CREATE USER 'root'@'db.server.com' IDENTIFIED BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO  'root'@'db.server.com' IDENTIFIED BY 'YOUR_PASSWORD' WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;

CREATE USER 'root'@'http.server.com' IDENTIFIED BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO  'root'@'http.server.com' IDENTIFIED BY 'YOUR_PASSWORD' WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;
```

#7. ISPConfig (Expert mode)
```sh
yum install -y perl-DateTime-Format-HTTP perl-DateTime-Format-Builder perl-Time*
service httpd restart
wget http://www.ispconfig.org/downloads/ISPConfig-3-stable.tar.gz
tar -zxvf ISPConfig-3-stable.tar.gz
sudo php -q ispconfig3_install/install/install.php
```