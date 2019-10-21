# ansible-letsencrypt-nginx

Using Ansible and Nginx to programmatically obtain Let's Encrypt SSL certificates on Ubuntu 18.

TLDR;

### Prerequisites

1. Provision a new machine from whichever provider you use (AWS, GCP, etc)
2. Setup your **A** / **CNAME** records to point to the **public ip address** of your newly created virtual machine.

### Steps

1. Ensure that you have hardened the machine for security.
2. Install Nginx on your new instance
3. Install Python SimpleJson that will be used later with Let's Encrypt tools
4. Install Let's Encrypt binaries via apt
5. Setup Nginx config to allow Let's Encrypt to access your machine via HTTP
6. Obtain SSL certification using Let's Encrpyt binaries
7. Add cron job to automatically renew SSL certificates, and reload Nginx when it does so
8. Have a great day and feel better about yourself
   <!-- TODO - Add mental health references -->

## Tutorial

Clone this repository into your local machine. Follow along once you have.

Note, you can fork/clone this repository. Check the `.gitignore` file for a list of files that will be ignored by default. Any file under `files/keys` will never be committed as it would be a terrible thing, had that not been the case.

#### 1. Setup your hosts file / inventory

The hosts file / inventory is the Ansible way of keeping track of IP addresses of your servers. In this repository the file is named `hosts`. Square brackets `[]` are used to create groups of servers that you can then refer to by name inside Ansible playbooks.

```
54.169.120.36

[webservers]
54.169.120.36
```

Replace the IP address with the public IP address of your newly created instance.

#### 2. Apply security rules to newly created instance

1. [Create a new SSH key pair](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) that you will use to login to machine once it is secured. For this example, it has been named `testingordinarysystems` (private key) and `testingordinarysystems.pub` (public key). Add these files to `files/keys`.
2. Generate a password hash of the password you wish to use for `sudo` in the new machine. You can run `mkpasswd --method=SHA-512 --stdin` in your local console for the same.
3. Set the appropriate values for variables `login_ssh_public_key_filename` (= `testingordinarysystems.pub` in this example) and `hashed_deploy_user_password` in `secure_server.yml`
4. Leave the variables `has_deployment_keys` and `has_ssh_config` with `false` for now.

Now, run the `secure_server` playbook using the command below.

```console
$ ansible-playbook -i hosts secure_server.yml
```

You should see a message like this

```
The authenticity of host '54.169.120.36 (54.169.120.36)' can't be established.
ECDSA key fingerprint is SHA256:......
Are you sure you want to continue connecting (yes/no)?
```

Answer yes. This will add the public key of the server to your `known_hosts` file. This is a one time operation. Future runs will not prompt you again.

At this point, in a few seconds, the rest of the process should fail miserably. The point being to get a bit more familiar with the possible errors that you may encounter down the line.

The error should be something along the lines of

```
fatal: [54.169.120.36]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: Warning: Permanently added '54.169.120.36' (ECDSA) to the list of known hosts.\r\nReceived disconnect from 54.169.120.36 port 22:2: Too many authentication failures\r\nDisconnected from 54.169.120.36 port 22", "unreachable": true}
```

This is Ansible telling you that it needs additional information to be able to connect to you newly created machine.

Note: there's more to this error. If you're interested check out [this superuser answer](https://superuser.com/a/187790)

On AWS, when you launch a new instance, it will prompt you to either create a new key pair for accessing your instance or choose from an existing one. If you're creating a new key pair, download the private key file (ends in `.pem`), save the file to `files/keys`.

Within your `hosts` file, make the following changes. Make sure you get the file names right.

```
54.169.120.36 ansible_user=ubuntu ansible_ssh_private_key_file=./files/keys/testing.pem ansible_ssh_common_args='-o IdentitiesOnly=yes'

[webservers]
54.169.120.36 ansible_user=ubuntu ansible_ssh_private_key_file=./files/keys/testing.pem ansible_ssh_common_args='-o IdentitiesOnly=yes'
```

Run the playbook again.

```console
$ ansible-playbook -i hosts secure_server.yml
```

A new error should greet you.

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0664 for 'testing.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "testing.pem": bad permissions
```

Fix it by changing the file permissions on your downloaded `pem` file. The filename of your key will be different ofcourse.

```console
$ chmod 400 ./files/keys/testing.pem
```

Run the playbook again.

```console
$ ansible-playbook -i hosts secure_server.yml
```

You should see an output with a list of actions that the playbook has peformed on your machine.

```console
PLAY [webservers] ****************************************************************************************************

TASK [secure_server : Update and upgrade apt packages] ***************************************************************
 [WARNING]: The value True (type bool) in a string field was converted to 'True' (type string). If this does not look
like what you expect, quote the entire value to ensure it does not change.

 [WARNING]: Could not find aptitude. Using apt-get instead

changed: [54.169.120.36]

TASK [secure_server : Ensure deploy user present] ********************************************************************
changed: [54.169.120.36]

TASK [secure_server : Set authorized key taken from file] ************************************************************
changed: [54.169.120.36]

TASK [secure_server : Set permissions on /home/deploy/.ssh/authorized_keys] ******************************************
changed: [54.169.120.36]

TASK [secure_server : include_tasks] *********************************************************************************
skipping: [54.169.120.36]

TASK [secure_server : include_tasks] *********************************************************************************
skipping: [54.169.120.36]

TASK [secure_server : Disable admin group in sudoers] ****************************************************************
changed: [54.169.120.36]

TASK [secure_server : Disable sudo group in sudoers] *****************************************************************
changed: [54.169.120.36]

TASK [secure_server : Add deploy to sudoers] *************************************************************************
changed: [54.169.120.36]

TASK [secure_server : Disable root login via password] ***************************************************************
changed: [54.169.120.36]

TASK [secure_server : Disable PasswordAuthentication] ****************************************************************
ok: [54.169.120.36]

TASK [secure_server : Install unattended-upgrades package] ***********************************************************
ok: [54.169.120.36]

TASK [secure_server : Enable periodic updates] ***********************************************************************
changed: [54.169.120.36]

TASK [secure_server : Enable ufw access for OpenSSH] *****************************************************************
changed: [54.169.120.36]

TASK [secure_server : Enable ufw] ************************************************************************************
changed: [54.169.120.36]

TASK [secure_server : Unconditionally reboot the machine after applying the updates] *********************************
changed: [54.169.120.36]

PLAY RECAP ***********************************************************************************************************
54.169.120.36              : ok=14   changed=12   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

The playbook has been successfully executed. Your server has some basic security in place now. You can now login to your machine using this command

```console
$ ssh deploy@54.169.120.36 -i ./files/keys/testingordinarysystems
```

It is definitely cumbersome to have to point to a particular ssh key everytime you want to login to your server. To make it easier, add the lines below to your ssh `config` file. If it doesn't exist, make one using the command below.

```console
$ touch ~/.ssh/config
```

Add these lines to the file using your editor of choice. Replace the ip address with the ip address of your machine. Replace the `IdentityFile` with the absolute path to your ssh private key (the one without `.pub` at the end).

```
Host 54.169.120.36
 PreferredAuthentications publickey
 IdentityFile /home/ordinarysystems/Documents/github/ansible-letsencrypt-nginx/files/keys/testingordinarysystems
 IdentitiesOnly yes
```

You can now login to your machine using the command

```console
$ ssh deploy@54.169.120.36
```

#### (Optional) Adding deployment keys to your machine

If this machine is going to be used to pull private repositories from GitHub/GitLab, etc, you can pre-setup the deployment keys necessary for the same.

Create a new SSH key pair, and add the public key to your list of authorized keys on GitHub/GitLab (out of scope for this tutorial). Name them `deploy` and `deploy.pub`. Place these files under `files/keys`.

Modify the file `files/keys/ssh_config`. Replace the `Host` and `HostName` parameters to values appropriate for your git provider. The example file assumes you use GitLab.

Set the parameters `has_deployment_keys` and `has_ssh_config` to `true` in `secure_server.yml`

Run the playbook again. Note, all the commands in the play are idempotent. This means you can run them any number of times, and the output will remain unchanged if nothing needs to be changed.

```console
$ ansible-playbook -i hosts secure_server.yml
```

You should see output similar to the one below

```
PLAY [webservers] ****************************************************************************************************

TASK [secure_server : Update and upgrade apt packages] ***************************************************************
 [WARNING]: The value True (type bool) in a string field was converted to 'True' (type string). If this does not look
like what you expect, quote the entire value to ensure it does not change.

 [WARNING]: Could not find aptitude. Using apt-get instead

ok: [54.169.120.36]

TASK [secure_server : Ensure deploy user present] ********************************************************************
ok: [54.169.120.36]

TASK [secure_server : Set authorized key taken from file] ************************************************************
ok: [54.169.120.36]

TASK [secure_server : Set permissions on /home/deploy/.ssh/authorized_keys] ******************************************
ok: [54.169.120.36]

TASK [secure_server : include_tasks] *********************************************************************************
included: /home/ordinarysystems/Documents/github/ansible-letsencrypt-nginx/roles/secure_server/tasks/copy_deployment_keys.yml for 54.169.120.36

TASK [secure_server : Copy deployment private key] *******************************************************************
changed: [54.169.120.36]

TASK [secure_server : Copy deployment public key] ********************************************************************
changed: [54.169.120.36]

TASK [secure_server : include_tasks] *********************************************************************************
included: /home/ordinarysystems/Documents/github/ansible-letsencrypt-nginx/roles/secure_server/tasks/copy_ssh_config.yml for 54.169.120.36

TASK [secure_server : Copy ssh config file] **************************************************************************
changed: [54.169.120.36]

TASK [secure_server : Disable admin group in sudoers] ****************************************************************
ok: [54.169.120.36]

TASK [secure_server : Disable sudo group in sudoers] *****************************************************************
ok: [54.169.120.36]

TASK [secure_server : Add deploy to sudoers] *************************************************************************
ok: [54.169.120.36]

TASK [secure_server : Disable root login via password] ***************************************************************
ok: [54.169.120.36]

TASK [secure_server : Disable PasswordAuthentication] ****************************************************************
ok: [54.169.120.36]

TASK [secure_server : Install unattended-upgrades package] ***********************************************************
ok: [54.169.120.36]

TASK [secure_server : Enable periodic updates] ***********************************************************************
ok: [54.169.120.36]

TASK [secure_server : Enable ufw access for OpenSSH] *****************************************************************
ok: [54.169.120.36]

TASK [secure_server : Enable ufw] ************************************************************************************
ok: [54.169.120.36]

TASK [secure_server : Unconditionally reboot the machine after applying the updates] *********************************
changed: [54.169.120.36]

PLAY RECAP ***********************************************************************************************************
54.169.120.36              : ok=19   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

You should now be able to pull your git repositories to this machine.

## Quick Links

#### What Is DNS? | How DNS Works

For a primer on what **A** / **CNAME** records actually are and do, I highly recommend going through [What Is DNS? | How DNS Works](https://www.cloudflare.com/learning/dns/what-is-dns/) by Cloudflare.

#### Nginx

If you're new to Nginx, checkout the [beginner's guide](https://nginx.org/en/docs/beginners_guide.html). You are not required to go through them as all the necessary configuration files required for this process are provided in this repository. It is ,however, highly recommended that you have a basic familiarity with the tools that you use.

#### Python simplejson

simplejson is a simple, fast, complete, correct and extensible JSON <http://json.org> encoder and decoder for Python 2.5+ and Python 3.3+.

- [simplejson @ pypi](https://pypi.org/project/simplejson/)
- [Official documentation](https://simplejson.readthedocs.io/en/latest/#module-simplejson)
- [GitHub repository](https://github.com/simplejson/simplejson)

#### Let's Encrypt

Let’s Encrypt is a free, automated, and open certificate authority (CA), run for the public’s benefit. It is a service provided by the [Internet Security Research Group (ISRG)](https://www.abetterinternet.org/).

- [Documentation](https://letsencrypt.org/docs/)

#### Setting up Ansible inside a virtual environment

- [Setup Ansible with Python Virtualenv](https://buildahomelab.com/2018/10/06/setup-ansible-with-virtualenv/)

#### Ansible stuff

- Hosts file / Inventory
- Ansible playbooks
- Ansible roles

#### SSH Stuff

- known hosts

#### Common errors

- https://superuser.com/questions/187779/too-many-authentication-failures-for-username
