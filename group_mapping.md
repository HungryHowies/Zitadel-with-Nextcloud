### Group Mapping 
Create a role in NextCloud project by clicking on the Project  NextCloud. On the left pane click on Roles.
Click the new button  and add the following.
```
Key = Groups
Display Name = Groups
Group = admin
```
Save and close.

In NextCloud Web UI under SSO & SAML  section scroll down to "Attribute mapping".
* Attribute to map the users groups to:
  
  Groups
* Group Mapping Prefix, default: SAML:
  
  admin
