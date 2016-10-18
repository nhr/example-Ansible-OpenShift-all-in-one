$ ansible-playbook devenv-launch.yml
=========================

## Ansible playbook for provisioning OpenShift All-In-One Using AWS
(for testing/development)

This is an example Ansible playbook that:

1. Launches an EC2 instance 
2. Starts an OpenShift All-In-One cluster in the EC2 
3. Deploys a sample cakephp-mysql app in project 'myproject'
4. Can be configured to run any post-cluster plays you need

###Initial Preparation Local Machine:
*  Clone this repository:
```
git clone --recursive git@github.com:sallyom/example-Ansible-OpenShift-all-in-one.git
```
*  sudo dnf install ansible
*  sudo pip install boto
*  export MY_LOCAL_PATH_HERE=/your/local/path/to/this/playbook
*  export INSTANCE_NAME=name your ec2 instance/if no export default '$USER-devenv' is given
*  need this file:
```
$cat ~/.aws/credentials
    [default]
    aws_access_key_id = XXXX
    aws_secret_access_key = XXXXX
```
* Fill in your `RedHat Subscription username/password and Pool ID` for `OpenShift Container Platform` in `vars/user_vars.yml`.
* You need to fill in `vars/launch_vars.yml`:
  * This setup assumes you have an AWS account and a VPC that enables public DNS hostnames.  See the **VPC Dashboard, Start VPC Wizard** link in the AWS console for more info.

Then run this:
### $ ansible-playbook devenv-launch.yml
Annoying caveat!! This will fail first, until you ssh into your instance and configure root login, like so:
```
ssh -i /path/to/sshkey ec2-user@<ec2-xx-xx-xx.compute-1.amazonaws.com>
vi /etc/ssh/sshd_config
  * Add this line: 'PermitRootLogin without-password' and close
exit
then:
vi ~/.ssh/authorized_keys
  * Remove everything before 'ssh-rsa' (command, no-port-forwarding...warn users...etc)
then exit out of instance and re-run:

$ ansible-playbook devenv-launch.yml

The playbook will re-use the instance you have configured
for root login.  Let me know if you can configure root login in a more efficient way, thanks!
```
All tasks are defined in roles/<task>/tasks/main.yml

## Now What?:
* You can run oc commands locally OR you can ssh into your ec2 and run oc commands there (you are logged in as 'system:admin' in your ec2 instance.  You can login as 'admin/password' on your local machine. `oc login -u admin -p password <public_dns_of _ec2>:8443`

* Tags and Tasks can be listed like so:
  
  `ansible-playbook devenv-launch.yml --list-tags`
  
  `ansible-playbook devenv-launch.yml --list-tasks`

* To access the web console: 
```
https://<ec2----.compute-1.amazonaws.com>:8443
 
login using 'anypassword' to gain admin access to all projects.

login with 'admin/password'
```

* Running the playbook a second time will skip tasks that have been done and are unchanged, so 
  a new ec2 instance will not be started every time the playbook is run.
* To terminate and start again, 
    * In AWS console, terminate the instance.
    * Now you are ready to run `$ ansible-playbook devenvup.yml` again!

