# Zitadel-with-Nextcloud

Installing and configuring Nextcloud  to authenacate with SAML against Zitadel

## Overview

The following documentation discribes the How-to for installing nextcloud (Lastest version) and configurations needed for HTTPS and the connection to authenicate against Zitadel. This setup is a Ubuntu-Server core installation, this means it has the minimum amount packages and/or depenedncies (i.e., install as you go). Its easier to install the correct packages first, then to  disable/remove old packages. Apache2 and MariaDB-Server will be installed along with setting up PHP-8.2.

## Prerequisite

* Ubuntu 22.04 LTS (recommended)
* MySQL 8.0+ or MariaDB 10.3/10.4/10.5/10.6 (recommended)
* PHP Runtime 8.2 (recommended) 8.3
* Completed all updates & upgrades.
* Static IP Address/DNS
* Fully qualified Domain name
* Set Date/Time.
* Set Hosts/Hostname

## PHP

Install PHP 8.2 on Ubuntu 22.04. By defualt Ubuntu does not install the correct verion of PHP needed so this section is a How-To.
need to redo  proceedure --> https://docs.nextcloud.com/server/latest/admin_manual/installation/command_line_installation.html
Adding the official PHP repository on Ubuntu.

```
apt-get install software-properties-common
```
```
sudo add-apt-repository ppa:ondrej/php
```
Press the **Enter** key when the system prompts you to. This will allow the system to use the repository as a source for new software.

installation of PHP 8.2 on Ubuntu 22.04 using this command.

Update Repository

```
apt-get update
```

PHP install

```
sudo apt-get install php8.2
```

If there are multiple PHP version installed this command will let you  set the correct PHP version. 

```
Configuring the operating system (OS) to use the new PHP version
```

Verifying the PHP version

```
php -v
```

Ubuntu 22.04 uses a few different commands to help manage Apache modules, so it utilizes a specific PHP version depending on which module is loaded.
This can be viewed  by running the following command.

```
ls /etc/apache2/mods-available/php*
```

The default LAMP stack install would have PHP 7.0 and the new PHP 8.2 install we just made, but running the next command shows that PHP 7.0 is still active.

```
 ls /etc/apache2/mods-available/php*
```

If there are older PHP 7.0 version using the following command.

```
sudo a2dismod php7.0
```

Then enable PHP 8.2 using this command.

```
sudo a2enmod php8.2
```

## Install Dependencies

Install the required dependencies. This may take  few minutes because it shoudl show it will upgrade to PHP-8.3.

```
sudo apt-get install php-xml
```

## Create Nextloud Database/User

Login

```
sudo mysql
```

Create User.

```
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
```

Create database and settings.

```
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

Grant Privileges.

```
GRANT ALL PRIVILEGES ON nextcloud.* TO 'username'@'localhost';
```

Reload the in-memory copy of privileges from the privilege tables.

```
FLUSH PRIVILEGES;
```

Exit.

```
exit
```

## Install Nextcloud 

```
wget https://download.nextcloud.com/server/releases/latest.tar.bz2
```

Extract Nextcloud package

```
tar -jxpvf latest.tar.bz2 -C /var/www/
```

Set Permissions.

```
sudo chown -R www-data:www-data /var/www/nextcloud
```
## Extra's
sudo phpenmod bcmath gmp imagick intl
sudo apt install unzip
sudo a2enmod dir env headers mime rewrite ssl



Install Let's encrypt.

```
sudo apt install certbot python3-certbot-apache
```

Create Certificates.

```
sudo certbot --apache
```

Restart Apache2

```
systemctl restart apache2
```

Disable site

```
sudo a2dissite 000-default.conf
```

Create NextCloud site and enable it.

```
vi /etc/apache2/sites-available/nextcloud.conf
```

configure Nextcloud Site.

```
<VirtualHost *:80>
      ServerName nextcloud.domain.com
      Redirect / https://nextcloud.domain.com/
</VirtualHost>
<IfModule mod_ssl.c>
   <VirtualHost *:443>
        ServerAdmin webmaster@localhost
        DocumentRoot "/var/www/nextcloud"
      <Directory "/var/www/nextcloud">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        SSLEngine on
        ServerName nextcloud.domain.com
        SSLCertificateFile /etc/letsencrypt/live/nextcloud.domain.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/nextcloud.domain.com/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
```

Set permisions.

```
sudo chown -R www-data:www-data /etc/apache2/sites-available/nextcloud.conf
```

Enable Nextlcoud site

```
sudo a2ensite nextcloud
```


## Nextcloud SSO Web UI configuration

Login to Nextcloud as the system administrator.

Prepare your Nextcloud instance for SSO & SAML Authentication. Enable APP called **SSO & SAML Authentication**.

Navigate to Administrator settings and left pane scroll down to **SSO & SAML Authentication**.

Following setting need to be configured.

  * Allow the use of multiple user back-ends (e.g. LDAP) = Enable
  * Attribute to map the UID = UserName
    
-------------Identity Provider Data---------------
  * https://zabbix-4rkfxx.zitadel.cloud/saml/v2/metadata
  * https://zabbix-4rkfxx.zitadel.cloud/saml/v2/SSO
  * https://zabbix-4rkfxx.zitadel.cloud/saml/v2/SLO

Insert Zitadel Key.
!!!!  Discrobe how-to get from zitadel 
```
-- Public X.509 certificate of the idp ---
MIIFIjCCAwqgAwIBAgICAxwwDQYJKoZIhvcNAQELBQAwLDEQMA4GA1UEChMHWklUQURFTDEYMBYGA1UEAxMPWklUQURFTC
BTQU1MIENBMB4XDTIzMTAxMzAxMTYyMloXDTI0MTAxMjA3MTYyMlowMjEQMA4GA1UEChMHWklUQURFTDEeMBwGA1UEAxMV
WklUQURFTCBTQU1MIHJlc3BvbnNlMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA4rFWxS87FCZrK2Uu1Qbb3t
DESBtUNKaynT34L1L+52YUJs+L3mz+PTQRNVFz2SIgfm5+BN4CQm+gpuzQUz2jOxsHs/zSTUEWMM4womyHgAiEwKrv+Bff
z1FKEEZZIqdKu/vLj0HwSnu22MLdd0oNppspd+h50Rhlz6UGdGkgVSAUWXKe4jX1TiSdXxLPYH8q7OKfdM0Yh5WNGxpTU5
LNY9aj4cuLHhA2/7m4F7fAvTGyicqVrX1EoymCc+nSdTBJi09vYABcoM1Vi4BQCx/gQWaBQZYvSmRkMMJXCe6NUf+WBrXO
xvqfJsvnGDp7atOfsaOtKeck5iKbWRjGPiTodsQ/p9ckzcsUB5eVORxnUlJBP6wUl46RjeoBe2QPExxe8otS/UWBO0IXGp
joOyzJc1Yg+Tc0GAUvwgPZ1FuUO1cvzoC8Atc2FqYBF9ycBOQPPfFQPA9WD4/EWjAFWr1OY3Jfb4/A/kHN7XsP/CL5o+3e
qCmHRQoENkUatL+5lrBxlhNm6Wa4n9gilfB58OuZneMOvL2GFqs5s6DKX1fn+SQul6+IqaMGZ597Nw7OHK9xY7Rie5JWDn
4kJPn8L40F6doo7mEQybncJiFpqqUe4ogihEHUlrR7f6MGpW5XxuGmqZW8KH99occELKY0UoVmdR/riBbzOnWr2YKz7dYd
XDcCAwEAAaNIMEYwDgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMCMB8GA1UdIwQYMBaAFK9UG6gHm1cFBE
i+sbhIgjsNrqbuMA0GCSqGSIb3DQEBCwUAA4ICAQCG8iIP+ocO8VBtW4SqVPYXsUpkzJlrNzECSuAZPRV2e+vndDMo028a
35Jd1I/mDDfoLsUUX+ekfQT7Vry/o0TYfpzCWhjWoc1Lwqr6psCKLY4klvuyOPtud7EPUQsukLivCQXhUW0sxrxig2ZQtJ
mQ89nU6Dg/HKNEZ2Qnhm5fkk/FOZGqZfoxwPJ3sF5rQbEQpa88qj1IneseEuVOC5+b1ix8nJslW9ukWRbXpiaEPpkP1Z/V
31yIvwYGB+PDxryVumWwnQfCx5NrvzOpX4ommMOlxD73BqFFfixtSqOKA+R4JN+L/F+hWiOSj77Jp1cwo2liSoaG/P3aSU
ewvz1Sj1Ig7Td8Nq313MYMTwimzrm8e8gxLRrosWAKcuIrz+NaabWLrbePrll2GvPUDYI4+Iwkf6YCyeLO9I5ff1O+cTs4
EaWuudlsip+55qJ6J0IIaOMagTu04UK6UTOpan9NKSvQSEFyooyGL+dSv8/WkOexEgy/62k41KlcjNMGc96MGOHleYtdWD
6HSF1B3WB+6MmNck7LSmSCm46v2wbNoQrSTBaiCHIEx0NFzTALG0ELdDAzivFKS9pBEPyK3McMWXCKYJjReTr5TRFE2FxT
aWnxeKtMn/UvE1I7PsgDcvm/a+TR48YW0k+yYegXktGeQxurA/aZqSjSBc55kfDR8A==
```

Bottom of the page click the **Download metadata XML** button, this will be uploaded to Zitadel Project called Nextcloud.
You should also see a Green metadata valid next to download.


## Zitadel

Create a project called nextcloud, create an application called nextcloud using SAML.
Click the button called Upload Metadata XML and upload the XML file from nextcloud.
Example:
```
<?xml version="1.0"?>
      <md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"
          validUntil="2023-10-26T03:08:08Z"
          cacheDuration="PT604800S"
          entityID="https://nextcloud.domain.com/index.php/apps/user_saml/saml/metadata">
<md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false"
        protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
<md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
          Location="https://nextcloud.domain.com/index.php/apps/user_saml/saml/sls" />
<md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameidformat:unspecified</md:NameIDFormat>
<md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
      Location="https://nextcloud.domain.com/index.php/apps/user_saml/saml/acs"
index="1" />
  </md:SPSSODescriptor>
</md:EntityDescriptor>
```
## Additional Notes:

Nextcloud Admin Manual.

https://docs.nextcloud.com/server/latest/admin_manual/installation/example_ubuntu.html
https://docs.nextcloud.com/server/latest/admin_manual/installation/system_requirements.html#server
https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html
https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/index.html























