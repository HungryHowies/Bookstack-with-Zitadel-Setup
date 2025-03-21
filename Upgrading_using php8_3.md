Upgrading issue  when bookstack requires php8.2. In this documentation Im will install php8.3 and upgrade composer.

Navigate to home directory of bookstack
```
cd /var/www/bookstack
```
```
sudo apt update
```
```
sudo apt install software-properties-common
```
```
sudo add-apt-repository ppa:ondrej/php
```
```
sudo apt update
```
```
sudo apt install -y php8.3 php8.3-curl php8.3-mbstring php8.3-ldap php8.3-xml php8.3-zip php8.3-gd php8.3-mysql libapache2-mod-php8.3
```
```
sudo a2dismod php7.4 php8.0 php8.1 php8.2
```
```
sudo a2enmod php8.3
```
```
sudo systemctl restart apache2
```

When Completed upgrade composer.
```
sudo composer self-update
```
