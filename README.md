# Configuring Nextcloud with DebOps

## Introduction

[DebOps](https://github.com/debops/debops) is a pretty extensive collection of [ansible](https://www.ansible.com/) playbooks with a large [documentation](https://docs.debops.org/en/master/).

It's used to configure, deploy and maintain any Debian based virtual machine.

DebOps offers, out of the box, hundreds of services well configured accordingly to the best practices.

If DebOps doesn't have a recipe for a specific software the user has the chance to add her own recipes.

## Audience

This repo is meant for devops/Ansible/DebOps beginners and it can be used as a tutorial.

## Expected result

You'll quickly spin up a Debian based virtual machine running Nextcloud, complete with LetsEncrypt certificates and mysql database.

> [Nextcloud](http://nextcloud.org/) is like a Google Drive on steroids. It has much more functionalities and, above all, is private. You master it.

## Assumptions

This repo assumes that:

* you are running a Debian personal computer (stable or testing vers)
* you own a domain like `example.com`
* you can manage the DNS for the domain above
* you have an account in a cloud hosting company like [Linode](https://www.linode.com/), [Digital Ocean](https://digitalocean.com), [Siteground](https://www.siteground.com/) and many others.
* you know how to create a clean debian virtual machine in your cloud provider
* you know how to add your ssh-public key to the virtual machine
* you will call your virtual machine `linda`
* you will access your Nextcloud at the address: https://nc.example.com

If you don't want to call your server "linda" then you have to change `linda` to `whatever-you-chose` anywhere in these configuration files.

## Install DebOps

Configure your environment so that `~/.local/bin` is in your PATH:

``` bash
mkdir -p ${HOME}/.local/bin &> /dev/null
PATH="${HOME}/.local/bin/:${PATH}"
```

Check your `~/.bashrc` file to be sure that PATH will retain the change after rebooting your pc.

Install these packages on your personal computer:

``` bash
sudo apt install ansible python3-future python3-ldap python3-netaddr python3-dnspython python3-passlib python3-pip

pip3 install --user debops
```

This transforms your pc in an _"ansible controller"_ meaning that you'll manage your virtual machines from this computer.

## Start

### step 1
Create the `linda` virtual machine via your cloud hosting company interface.

### step 2

Add _linda.example.com_ and _nc.example.com_ as "A records" of your DNS.

### step 3
Edit your personal computer `/etc/hosts` file like this:

``` bash
<ip-of-your-vm> linda.example.com linda
```

### step 4
Clone this repo on your pc:

``` bash
git clone https://github.com/19hundreds/nextcloud_with_debops.git
```

### step 5
Edit these files accordingly to the side notes:

``` bash
├── ansible
│   ├── inventory
│   │   ├── group_vars
│   │   │   └── all
│   │   │       ├── bootstrap.yml       # do not touch
│   │   │       ├── netbase.yml         # change example.com with your domain
│   │   │       ├── ntp.yml             # change timezone if you like
│   │   │       └── sshd.yml            # change ssh IP whitelist (see below)
│   │   ├── hosts                       # change example.com with your domain
│   │   └── host_vars
│   │       ├── linda
│   │       │   ├── bootstrap.yml       # change example.com with your domain
│   │       │   ├── mariadb_server.yml  # change email and back-up dir as you like
│   │       │   └── owncloud.yml        # do not touch
│   │       └────── pki.yml             # change example.com with your domain
│   ├── playbooks                       # do not touch
│   ├── roles                           # do not touch
│   └── secret                          # do not touch
├── ansible-full.cfg                    # do not touch (it's here as example)
├── LICENSE
└── README.md
```

### step 6

Check if you can login via ssh in your `linda`

``` bash
ssh root@linda
```

If you can't, fix it and then go ahead.

### step 7

Open a terminal in your pc and:

``` bash
cd nextcloud_with_debops

debops
```

Let it run. It takes a long time, like an hour (but it depends a lot on several factors). Quality comes at a price :-)

DebOps takes care of an enormous amount of things that non-professional system administrator usually overlooks. Additionally, installing NextCloud involves the configuration of several services therefore the waiting time is an irrelevant cost compared to the advantages.

When it's done, check if you have any error. You should have no issues otherwise google for solutions before going ahead.

### step 8

Make LetsEncrypt certificates permanent and consistent by changing the `pky.yml` file:

``` bash
    #acme_ca: 'le-staging-v2' #for testing
    acme_ca: 'le-live-v2'      #for production
```

Run `debops` again:

``` bash
debops
```

At the end you should be able to access your Nextcloud instance at http://nc.example.com