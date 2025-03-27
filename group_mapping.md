This documentation will describe how to set a SSO user this Super Admin rights. There will be two Authorization/s. One Authorization is for the project called NextCloud and the other Authorization will be added to that user. 

Group Mapping 

Navigate to the Nextcloud Project in Zitadel.

On the left pane click on Roles

Click the New button and add the following.
```
Key = Admins
Display Name = admin
Group = admin
```


Save and close.

Project  Authorization

On the left pane click on Authorizations and then click new button.

Select the user for access to NextCloud and click continue.

Select the role Admins for that user by checking the tic box. Then click the Save button.

Navigate the home page of the project NextCloud. Under Settings section in the center pane ensure that the Assert Roles on Authentication tic box is enabled.

Zitadel User Metadata

Click on the user that should have Admin rights. On the left pane click on Metadata.

Add the following.

```
Key = Admins
Value = admin
```

Users Authorization

Click on Authorizations on the left pane, The Role Admins should be shown at this time for the SSO user for NextCloud.

Nextcloud User Groups Attributes

In NextCloud Web UI under SSO & SAML  section scroll down to "Attribute mapping".

* Attribute to map the users groups to: Admins
