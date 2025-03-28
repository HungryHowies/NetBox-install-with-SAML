This documentation will how to configure Zitadel and Netbok and assign SSO a group attribute. This can be for any type of cgroup created in Netbox.

## Nebox

Log into Netbox web UI with administratory rights.

Left pane click on Admin.
Create permissions first by clicking on Permissions in the left pane.
Click the add button  top right and give the permission a name.
Select any or all action required.
Next select Object types that is needed. Then click create button.
On the left pane click on Groups and click the add button top right. and give the group a name.
Next to Permissions section select the drop down fromt eh permission create above. The click create.

## Zitadel

Choose the Project created for Netbox SSO.
Click on roles and configure. Add the following to that Role.

```
Key: netbox-admin
DisplayName: netbox-admin
Group: netbox-admin
```

Click on Autherorizations in the Left pane of that Project.Click the New button.
Select the user needed. Click Continue.
Click the tic box assocceated wwith the project Netbox role. Click Save button.
Navigate to the Users section on top pane.
In that users profile click Metadata on the left pane and add the following.

```
Key: netbox-admin
Value: netbox-admin
```
Once completed  click the save button.
Ensure the Settings in the Project Netbox  the tic  box for Assert Roles on Authentication is enabled.





 
 


 

 




