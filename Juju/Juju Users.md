## Juju User Management and Access Control
### Task 1 : Juju users
Identify the Current User:
```
whoami
```
List all Juju users:
```
juju users
```
Show details for the admin user:
```
juju show-user admin
```
Add a New User:
```
juju add-user testuser
```
List all users again to verify:
```
juju users
```
### Task 2 : Manage Access and Permissions
To understand how to grant different access levels to users in Juju, you can use the following command, which provides information about the various permissions you can assign.
```
juju grant --help
```
Grant read permission to `testuser` for the `k8s-model` model:
```
juju grant testuser read k8s-model
```
Note: Replace the model name as per your requirement
```
juju models --user testuser
```

### Task 3 : Verify User Access and Permissions
Change the password for the current user:
```
juju change-user-password
```
Change the password for the user `testuser`:
```
juju change-user-password testuser
```
Now logout from `admin` user, and login with  `testuser` to verify permissions:
```
juju logout
```
```
juju login -u testuser
```
check currently login user andList accessible models :
```
whoami
```
```
juju models
```
Now, Attempt to deploy any application i.e Prometheus or any other write actions.
```
juju deploy prometheus
```
 `testuser`  has only read permissions, this command will fail with a permission denied error.
 
 Logout and login back to the admin user
 ```
juju logout
```
```
juju login -u admin
```
 
