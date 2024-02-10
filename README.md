# Zitadel-with-Nextcloud

Installing and configuring Nextcloud  to authenticate using SAML with Zitadel.

## Overview

The following documentation describes the How-to for installing NextCloud (Latest version) and configurations needed for HTTPS and the connection to authenticate against Zitadel. This setup is an Ubuntu-Server core installation, this means it has the minimum amount packages and/or dependencies (i.e., install as you go). It’s easier to install the correct packages first, then to disable/remove old packages. Apache2 and MariaDB-Server will be installed along with setting up PHP-8.2. MySQL 8.0+ or MariaDB 10.3/10.4/10.5/10.6 (recommended)

## Prerequisite

* Ubuntu 22.04 LTS (recommended)
* Completed all updates & upgrades.
* Static IP Address/DNS
* Fully qualified Domain name
* Set Date/Time.
* Set Hosts/Hostname

## PHP 

Install PHP 8.2 on Ubuntu 22.04. By default, Ubuntu does not install the correct version of PHP needed so this section is a How-To.

Adding the official PHP repository on Ubuntu.

```
apt-get update && apt-get clean && apt-get install -y build-essential
```
```
sudo add-apt-repository ppa:ondrej/php
```
Press the **Enter** key when the system prompts you to. This will allow the system to use the repository as a source for new software.

Install PHP 8.2 on Ubuntu 22.04.

Update Repository

```
apt-get update
```

PHP install.

```
sudo apt-get install php8.2
```

Then enable PHP 8.2 using this command.

```
sudo a2enmod php8.2
```

Verifying the PHP version

```
php -v
```

Ubuntu 22.04 uses a few different commands to help manage Apache modules, so it utilizes a specific PHP version depending on which module is loaded.
This can be viewed by running the following command.

```
ls /etc/apache2/mods-available/php*
```

The default LAMP stack install might have PHP 7.0 and the new version PHP 8.2 install. 
Check all PHP versions installed on ubuntu by running the below commands.

```
dpkg-query -l | grep -i php
```

NOTE: If there are older verions of PHP installed, might want to disable then by following command. 

Example:

```
sudo a2dismod php7.0
```

## Install Dependencies

Install the required dependencies for Nextcloud.

```
apt install  php8.2-bz2 php8.2-gd php8.2-mysql php8.2-curl php8.2-mbstring php8.2-imagick php8.2-zip php8.2-common php8.2-curl php8.2-xml  php8.2-posix php8.2-bcmath php8.2-xml php8.2-intl php8.2-gmp zip unzip wget
```

Install Apache2 and MariaDb. 

```
apt install apache2 mariadb-server
```

## Create Nextcloud Database/User

Login

```
sudo mysql
```

Create Nextcloud database User.

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

Extract Nextcloud package and move it to WWW directory. 

```
tar -jxpvf latest.tar.bz2 -C /var/www/
```

Set Permissions.

```
sudo chown -R www-data:www-data /var/www/nextcloud
```

## PHP and Apache Mod's

Use phpenmod command followed by module name to enable specific PHP module on your system.

```
sudo phpenmod bcmath gmp imagick intl
```

Enable Apache Mod's

```
sudo a2enmod dir env headers mime rewrite ssl
```


## Install Let's encrypt.

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

Disable 000-default.conf  site.

```
sudo a2dissite 000-default.conf
```

Change directory.

```
cd /etc/apache2/sites-available
```

Copy 000-default-le-ssl.conf to nextcloud.conf. This will preserve the configuration from Let's encrypt.

```
cp 000-default-le-ssl.conf nextcloud.conf
```

Disable site 000-default-le-ssl.conf.

```
sudo a2dissite 000-default-le-ssl.conf
```

Login to the NextCloud site file. Adjust configuration as needed.

```
vi /etc/apache2/sites-available/nextcloud.conf
```
Add following lines on top of nextcloud.conf file
```
<VirtualHost *:80>
      ServerName nextcloud.domain.com
      Redirect / https://nextcloud.domain.com/
</VirtualHost>
```

Adjust the line Documentroot section.

```
DocumentRoot /var/www/nextcloud
```

Add Directory section.

```
<Directory "/var/www/nextcloud">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```

Nextcloud Site example.

```
<VirtualHost *:80>
      ServerName nextcloud.domain.com
      Redirect / https://nextcloud.domain.com/
</VirtualHost>
   <VirtualHost *:443>
        ServerAdmin someuser@localhost
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

Set permissions on Nextcloud site file.

```
sudo chown -R www-data:www-data /etc/apache2/sites-available/nextcloud.conf
```

Enable Nextcloud site.

```
sudo a2ensite nextcloud
```

Use the occ command to complete your installation. This takes the place of running the graphical Installation Wizard.

Change Directory.

```
cd /var/www/nextcloud/
```

Execute OCC command and ensure the database and database user are correct. Before executing the following command.

```
sudo -u www-data php occ  maintenance:install \
--database='mysql' --database-name='nextcloud' \
--database-user=username --database-pass='password' \
--admin-user='admin' --admin-pass='password'
```

##  Nextcloud Config.php

This example "Localhost”  is not in use. Adjusting the config.php file from these two lines as needed.

Edit config.php file
```
vi /var/www/nextcloud/config/config.php
```

**Before** adjusting the config.php file.

```
'trusted_domains' =>
  array (
  		  0 => 'localhost',
```
```
'overwrite.cli.url' => 'http://localhost',
```

**After** adjusting the config.php file.

```
'trusted_domains' =>
  array (
    0 => 'localhost','nextcloud.domain.com',
```

```
'overwrite.cli.url' => 'https://nextcloud.domain.com',
```

Restart Apache2 service.


```
systemctl restart apache2
```

Login 

```
https://nextcloud.domain.com
```

## Nextcloud SSO Web UI configuration

Login to Nextcloud as the system administrator. 

Navigate to the Apps section, this would be located under username in upper right corner, click Apps.

![image](https://github.com/HungryHowies/Zitadel-with-Nextcloud/assets/22652276/60493504-42c8-4ff3-b060-9f4785ab898f)

Copy && Paste **SSO & SAML Authentication** in the search bar.

![image](https://github.com/HungryHowies/Zitadel-with-Nextcloud/assets/22652276/394f0f67-7938-4982-968d-055e4bb93959)


Download && Enable APP called **SSO & SAML Authentication**.

![image](https://github.com/HungryHowies/Zitadel-with-Nextcloud/assets/22652276/98791b47-3940-40b1-87ab-1888d3cbf516)


Navigate to Administrator settings and left pane scroll down to **SSO & SAML Authentication**.

![image](https://github.com/HungryHowies/Zitadel-with-Nextcloud/assets/22652276/f86a53b2-1bf6-4931-87b4-130393157b99)

Choose the Use built-in SAML authentication button.

![image](https://github.com/HungryHowies/Zitadel-with-Nextcloud/assets/22652276/ceff1cd7-2471-4054-8992-3ee483c1b095)

## Prepare your Nextcloud instance for SSO & SAML Authentication.
The following setting need to be configured. Enable the following.

  * Allow the use of multiple user back-ends (testing)
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
