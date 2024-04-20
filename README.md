# Bookstack-with-Zitadel-Setup

## Overview
Installing bookstack with Let's encrypt and connecting it to Zitadel for SSO using SAML application.

NOTE: This was tested on Zitadel Instance v2.37.3. There may be a bug on later version  when using ""Option 1. Specify the url where metadata file is located""

Prerequisite
  * Ubuntu 22.04.3 LTS
  * Completed all updates && upgrades
  * Set time zone
  * Apache install
  * Let's Encrypt
  * Zitadel instance
  * MySQL >= 5.7 or MariaDB >= 10.2
  * PHP 8.1 

## Bookstack install

```
wget https://raw.githubusercontent.com/BookStackApp/devops/main/scripts/installation-ubuntu-22.04.sh
```
```
chmod a+x installation-ubuntu-22.04.sh
```
```
sudo ./installation-ubuntu-22.04.sh
```
## Let's Encrypt

Certbot install
```
sudo apt install certbot python3-certbot-apache
```
Create certificates.
```
sudo certbot --apache
```
Apache  configuration

Ensure Apapche  Mods are installed.
```
sudo a2enmod dir env headers mime rewrite ssl
```
```
<VirtualHost *:80>
      ServerName bookstack.domain.com
      Redirect / https://bookstack.domain.com/
</VirtualHost>
    <IfModule mod_ssl.c>
      <VirtualHost *:443>
      ServerName bookstack.domain.com
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/bookstack/public/
      <Directory /var/www/bookstack/public/>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
    <IfModule mod_rewrite.c>
      <IfModule mod_negotiation.c>
          Options -MultiViews -Indexes
      </IfModule>
        RewriteEngine On
        RewriteCond %{HTTP:Authorization} .
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteCond %{REQUEST_URI} (.+)/$
        RewriteRule ^ %1 [L,R=301]
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^ index.php [L]
      </IfModule>
    </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
      SSLCertificateFile /etc/letsencrypt/live/bookstack.domain.com/fullchain.pem
      SSLCertificateKeyFile /etc/letsencrypt/live/bookstack.domain.com/privkey.pem
      Include /etc/letsencrypt/options-ssl-apache.conf
    </VirtualHost>
</IfModule>
```

Restart Apache Service

```
systemctl restart apache2
```
## Configured Bookstacks .env file

This is located in /var/www/bookstack.
Note: To get the x509 certificate. This is done by using the end point /saml/v2/metadata on the FQDN or IP Address on your Zitadel instance.

Example: 
```
https://zitadel.domain.com/saml/v2/metadata
```
The .env file configuration.

```
APP_KEY=base64:bADpkrGFLJUFcnZb1z8qWwvDewCpyCwh/VE964fL4gI=
APP_URL=https://bookstack.domain.com
DB_HOST=localhost
DB_DATABASE=bookstack
DB_USERNAME=bookstack
DB_PASSWORD=WbGlKXtdfzkn7
MAIL_DRIVER=smtp
MAIL_FROM_NAME="BookStack"
MAIL_FROM=bookstack@example.com
MAIL_HOST=localhost
MAIL_PORT=587
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
SAML2_IDP_ENTITYID=https://zitadel.domain.com/saml/v2/metadata
SAML2_AUTOLOAD_METADATA=false
SAML2_IDP_SSO=https://zitadel.domain.com/saml/v2/SSO
SAML2_IDP_SLO=https://zitadel.domain.com/saml/v2/SLO

SAML2_IDP_x509=MIIFIjCCAwqgAwIBAgICAxwwDQYJKoZIhvcNAQELBQAwLDEQMA4GA1UEChMHWklUQURFTDEYMBYGA1U
EAxMPWklUQURFTCBTQU1MIENBMB4XDTIzMTAxMzAxMTYyMloXDTI0MTAxMjA3MTYyMlowMjEQMA4GA1UEChMHWklUQURFT
DEeMBwGA1UEAxMVWklUQURFTCBTQU1MIHJlc3BvbnNlMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA4rFWxS8
7............................................................................................
eLO9I5ff1O+cTs4EaWuudlsip+55qJ6J0IIaOMagTu04UK6UTOpan9NKSvQSEFyooyGL+dSv8/WkOexEgy/62k41KlcjNM
Gc96MGOHleYtdWD6HSF1B3WB+6MmNck7LSmSCm46v2wbNoQrSTBaiCHIEx0NFzTALG0ELdDAzivFKS9pBEPyK3McMWXCKY
JjReTr5TRFE2FxTaWnxeKtMn/UvE1I7PsgDcvm/a+TR48YW0k+yYegXktGeQxurA/aZqSjSBc55kfDR8A==

SAML2_IDP_AUTHNCONTEXT=true
```
## Create a SAML Application/Project.

Bookstack can use a URL to insert the metadata needed for the SAML Application XML file as shown below.
 
```
https://bookstack.domain.com/saml2/metadata
```
![image](https://github.com/HungryHowies/Bookstack-with-Zitadel-Setup/assets/22652276/dee78558-7ddc-490d-b257-8585ec6b8123)

Click Continue and save.























