This documentation will show how to configure Zitadel and Netbox and then assign SSO a group attribute. This can be for any type of groups created in Netbox.

## Nebox

Log into Netbox web UI with administrator rights.

Left pane click on Admin.
Create permissions first by clicking on Permissions in the left pane.
Click the add button on the top right and give the permission a name.
Select any or all action required.
Next select Object types that are needed. Then click the create button.
On the left pane click on Groups and click the add button top right. and give the group a name.
Next to Permissions section select the drop down from the permission create above. Then click create button.

## Zitadel

Choose the Project created for Netbox SSO.
Click on roles and configure. Add the following to that Role.

```
Key: netbox-admin
DisplayName: netbox-admin
Group: netbox-admin
```

Click on Authorizations in the Left Pane of that Project. Click the New button.
Select the user needed. Click Continue.
Click the tic box associated with the project Netbox role. Click Save button.
Navigate to the Users section on top pane.
In that users profile click Metadata on the left pane and add the following.

```
Key: netbox-admin
Value: netbox-admin
```
Once complete click the save button.
Ensure the Settings in the Project Netbox  the tic  box for Assert Roles on Authentication is enabled.
