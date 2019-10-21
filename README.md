# ansible-letsencrypt-nginx

Using Ansible and Nginx to programmatically obtain Let's Encrypt SSL certificates on Ubuntu 18.

TLDR;

### Prerequisites

1. Provision a new machine from whichever provider you use (AWS, GCP, etc)
2. Ensure that you have hardened the machine for security. You can follow the steps outlined in [ansible-server-hardening](https://able.bio/jibiabraham/basic-server-hardening-using-ansible--127r833) for the same. Or follow your own security practices.
3. Setup your **A** / **CNAME** records to point to the **public ip address** of your newly created virtual machine.

### Steps

1. Install Nginx on your new instance
2. Install Python SimpleJson that will be used later with Let's Encrypt tools
3. Install Let's Encrypt binaries via apt
4. Setup Nginx config to allow Let's Encrypt to access your machine via HTTP
5. Obtain SSL certification using Let's Encrpyt binaries
6. Add cron job to automatically renew SSL certificates, and reload Nginx when it does so
7. Have a great day and feel better about yourself
   <!-- TODO - Add mental health references -->

## Tutorial

#### 1. Setup your hosts file / inventory

The hosts file / inventory is the Ansible way of keeping track of IP addresses of your servers. In this repository the file is named `hosts`. Square brackets `[]` are used to create groups of servers that you can then refer to by name inside Ansible playbooks.

```
192.168.1.42

[webservers]
192.168.1.42
```

Replace the IP address with the public IP address of your newly created instance.

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
