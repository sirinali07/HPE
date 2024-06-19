The `Juju Dashboard` provides an interface where you can view controllers and models, monitor deployment statuses, manage access permissions, execute actions, and configure applications. It supports the local Juju environment.

### Task 1 : Enable Ubuntu Desktop

Update Package Repositories and Install XRDP:
```
sudo apt-get update
```
```
sudo apt-get install xrdp -y
```
Install Ubuntu GNOME Desktop:
```
sudo apt-get install ubuntu-gnome-desktop -y
```

Enable and Restart XRDP Service:
```
sudo systemctl enable xrdp
```
```
sudo systemctl restart xrdp
```
```
sudo systemctl status xrdp
```

Set Password for Ubuntu User:
```
sudo passwd ubuntu
```

Note: Ensure RDP port 3389 is allowed in your AWS Security Group.

### Task 2: Setting Up Juju and Deploying Juju Dashboard

Access Juju Environment and Controller:
```
juju models
```
```
juju switch controller
```

Deploy Juju Dashboard:

```
juju deploy juju-dashboard
```

Integrate Juju Dashboard with Controller:

```
juju integrate juju-dashboard controller
```

Check Dashboard Status and Relations:

```
juju status --relations
```

Expose Juju Dashboard:

```
juju expose juju-dashboard
```

Accessing Juju Dashboard:

```
juju dashboard
```
The above command will display the `URL` and `login` credentials for accessing the Juju Dashboard.

### Task 3 : Accessing the Juju Dashboard via Web Browser (On Local Ubuntu VM)

* Open Browser on JuJu Server (Ubuntu Machine):

* Paste the URL provided by juju dashboard.

* Enter the username (admin) and the password provided.
