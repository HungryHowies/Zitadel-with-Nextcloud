### Group Mapping 

Create a role in Zitadel in NextCloud Project by clicking on the Project NextCloud. On the left pane click on Roles.
Click the new button and add the following.
```
Key = Admins
Display Name = admin
Group = admin
```

Save and close.



Navigate to Authorization section and click new button.

Add the user for access to nextcloud and click continue.

Select the role for that user by checking the tic box. The click the Save button.

Under Settings section ensure that the Assert Roles on Authentication tic box is enabled.


## Edit Zitadel User metadata.

Click on the user that should have Admin rights. On the left pane click on Metadata.

Add the following.
```
Key = Admins
Value = admin
```
## Users Authorization.

Click on Authorizations then click the New button and add the user to Nextcloud Project.

Authorizations in the Project called Nextcloud. 

## Nextcloud User Groups Attributes

In NextCloud Web UI under SSO & SAML  section scroll down to "Attribute mapping".

* Attribute to map the users groups to: Admins
