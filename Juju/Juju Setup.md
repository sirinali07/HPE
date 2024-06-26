## Setting Up Juju on Ubuntu EC2 Instance
`Prerequisites:`
* AWS Account with access to create EC2 instances
* Access Key ID and Secret Access Key for AWS

### Task 1: Launching Instances on AWS
* `1` Instances of `4 cores, 4GB RAM`, with `Storage: 20 GB` instance with OS version as `Ubuntu 20.04 LTS` in your preferred region 

Connect to your EC2 instance using SSH, and Set the Hostname 
```
sudo hostnamectl hostname Juju-Server
```
```
bash
```
### Task 2: Install Juju
Install Juju using snap:
```
sudo snap install juju --classic
```
Verify the installation:
```
juju version
```
Prepare Juju Environment:
```
mkdir -p ~/.local/share/juju/
```

Add AWS credentials to Juju:
```
juju add-credential aws
```
Follow the on-screen prompts to enter a credential name (e.g., aws-cred), select a region (e.g., us-west-1), and provide your AWS access key and secret key.
Verify credentials:
```
juju clouds
```
Bootstrap the Juju controller:

juju bootstrap <region> <controller-name> --credential <credential-name>

```
juju bootstrap aws/us-west-1 my-controller --credential aws-cred
```
Check the controller status:
```
juju controllers
```
Creating a Juju Model:

juju add-model <model-name>  --credential <credential-name>l aws-cred <region>

```
juju add-model app-model --credential aws-cred aws/us-west-1
```
List and verify  models:

```
juju models
```

### Task 3: Deploying Application with Juju:
Deploy the Apache2 application using Juju:
```
juju deploy apache2 --constraints "instance-type=t3.medium" -n 1
```
`Note:` This command deploys a single Apache2 unit using the more cost-effective "t3.medium" instance type. Adjust the constraints and number of units based on your needs.

Monitor the deployment status
```
juju status
```
Expose the Apache2 service to the internet:
```
juju expose apache2
```
Check deployment status again:
```
juju status
```
Access the Apache2 application in a web browser:
* Find the `public IP` of the machine where Apache2 is deployed from the `juju status` output.
* Ensure port 80 is open in the security group of the Apache2 machine.
* Access the Apache2 application in a web browser: `http://<Public-IP>`
  

