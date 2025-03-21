### Group Mapping 
Create a role in Zitadel in NextCloud Project by clicking on the Project NextCloud. On the left pane click on Roles.
Click the new button and add the following.
```
Key = Admins
Display Name = admin
Group = admin
```
Save and close.

Edit Zitadel User metadata.
Click on the user that should have Admin rights. On the left pane click on Metadata.
Add the following.
```
Key = Admins
Value = admin
```
Authorizations in the Project called Nextcloud. 
Click on New button and Select User needed. Click Continue then save.


In NextCloud Web UI under SSO & SAML  section scroll down to "Attribute mapping".
* Attribute to map the users groups to: Admins
