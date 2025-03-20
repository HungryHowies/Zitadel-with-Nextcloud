### Using DHCP for Nextcloud
In this documentation I’m going to explain the fix in case the IP Address changes when using DHCP. In this environment this does not have a FQDN, just an IP Addresses.

When using DHCP for a virtual machine and the IP address changes these are the steps needed to fix the Web UI connection. There are a couple files that need adjustment.

Login the Virtual machine as admin user.
After you execute the following command, may need to re-login if your using SSH.
```
sudo netplan apply
```
Show the IP address and copy it.
```
ip a
```
Configure apache2 configuration file
```
vi /etc/apache2/sites-available/nextcloud.conf
```
Edit the following lines. Take note, you may have two lines with ServerName for port  80 and Port 443.
```
ServerName <ip-address>
Redirect / https://<ip-address>
```

Edit the NextCloud php file.
```
vi /var/www/nextcloud/config/config.php
```
adjust the following lines.
```
array (
    0 => 'localhost',
    1 => ‘<ip-address>
  ),
'overwrite.cli.url' => 'https://<ip-address>,
```
Restart apache2 service.
```
Systemctl restart apache2
```
