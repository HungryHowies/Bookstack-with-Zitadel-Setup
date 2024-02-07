# Bookstack-with-Zitadel-Setup

## Overview
Installing bookstack with Let's encrypt and connecting it to Zitadel for SSO using SAML application.

Prerequisite
  * Ubuntu 22.04.3 LTS
  * Completed all updates && upgrades
  * Set time zone
  * Apache install
  * Let's Encrypt
  * Zitadel instance
  * MySQL 8.0
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
APP_URL=https://bookstack.domain.com.com
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
AUTH_METHOD=saml2
SAML2_NAME=SSO
SAML2_EMAIL_ATTRIBUTE=Email
SAML2_EXTERNAL_ID_ATTRIBUTE=uid
SAML2_DISPLAY_NAME_ATTRIBUTES=UserName
SAML2_IDP_ENTITYID=bookstack
SAML2_AUTOLOAD_METADATA=false
SAML2_IDP_SSO=https://zitadel.domain.com/saml/v2/SSO

SAML2_IDP_x509=MIIFIjCCAwqgAwIBAgICAxwwDQYJKoZIhvcNAQELBQAwLDEQMA4GA1UEChMHWklUQURFTDEYMBYGA1U
EAxMPWklUQURFTCBTQU1MIENBMB4XDTIzMTAxMzAxMTYyMloXDTI0MTAxMjA3MTYyMlowMjEQMA4GA1UEChMHWklUQURFT
DEeMBwGA1UEAxMVWklUQURFTCBTQU1MIHJlc3BvbnNlMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA4rFWxS8
7FCZrK2Uu1Qbb3tDESBtUNKaynT34L1L+52YUJs+L3mz+PTQRNVFz2SIgfm5+BN4CQm+gpuzQUz2jOxsHs/zSTUEWMM4wo
myHgAiEwKrv+Bffz1FKEEZZIqdKu/vLj0HwSnu22MLdd0oNppspd+h50Rhlz6UGdGkgVSAUWXKe4jX1TiSdXxLPYH8q7OK
fdM0Yh5WNGxpTU5LNY9aj4cuLHhA2/7m4F7fAvTGyicqVrX1EoymCc+nSdTBJi09vYABcoM1Vi4BQCx/gQWaBQZYvSmRkM
MJXCe6NUf+WBrXOxvqfJsvnGDp7atOfsaOtKeck5iKbWRjGPiTodsQ/p9ckzcsUB5eVORxnUlJBP6wUl46RjeoBe2QPExx
e8otS/UWBO0IXGpjoOyzJc1Yg+Tc0GAUvwgPZ1FuUO1cvzoC8Atc2FqYBF9ycBOQPPfFQPA9WD4/EWjAFWr1OY3Jfb4/A/
kHN7XsP/CL5o+3eqCmHRQoENkUatL+5lrBxlhNm6Wa4n9gilfB58OuZneMOvL2GFqs5s6DKX1fn+SQul6+IqaMGZ597Nw7
OHK9xY7Rie5JWDn4kJPn8L40F6doo7mEQybncJiFpqqUe4ogihEHUlrR7f6MGpW5XxuGmqZW8KH99occELKY0UoVmdR/ri
BbzOnWr2YKz7dYdXDcCAwEAAaNIMEYwDgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMCMB8GA1UdIwQYMBa
AFK9UG6gHm1cFBEi+sbhIgjsNrqbuMA0GCSqGSIb3DQEBCwUAA4ICAQCG8iIP+ocO8VBtW4SqVPYXsUpkzJlrNzECSuAZP
RV2e+vndDMo028a35Jd1I/mDDfoLsUUX+ekfQT7Vry/o0TYfpzCWhjWoc1Lwqr6psCKLY4klvuyOPtud7EPUQsukLivCQX
hUW0sxrxig2ZQtJmQ89nU6Dg/HKNEZ2Qnhm5fkk/FOZGqZfoxwPJ3sF5rQbEQpa88qj1IneseEuVOC5+b1ix8nJslW9ukW
RbXpiaEPpkP1Z/V31yIvwYGB+PDxryVumWwnQfCx5NrvzOpX4ommMOlxD73BqFFfixtSqOKA+R4JN+L/F+hWiOSj77Jp1c
wo2liSoaG/P3aSUewvz1Sj1Ig7Td8Nq313MYMTwimzrm8e8gxLRrosWAKcuIrz+NaabWLrbePrll2GvPUDYI4+Iwkf6YCy
eLO9I5ff1O+cTs4EaWuudlsip+55qJ6J0IIaOMagTu04UK6UTOpan9NKSvQSEFyooyGL+dSv8/WkOexEgy/62k41KlcjNM
Gc96MGOHleYtdWD6HSF1B3WB+6MmNck7LSmSCm46v2wbNoQrSTBaiCHIEx0NFzTALG0ELdDAzivFKS9pBEPyK3McMWXCKY
JjReTr5TRFE2FxTaWnxeKtMn/UvE1I7PsgDcvm/a+TR48YW0k+yYegXktGeQxurA/aZqSjSBc55kfDR8A==

SAML2_IDP_AUTHNCONTEXT=true
```
## Create a SAML Application/Project.

 Bookstack I can use a URL to insert the metadata need for the SAML Application.
 
```
https://bookstack.domain.com /saml2/metadata
```
![image](https://github.com/HungryHowies/Bookstack-with-Zitadel-Setup/assets/22652276/dee78558-7ddc-490d-b257-8585ec6b8123)

Click Continue and save.























