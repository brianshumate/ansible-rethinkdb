# RethinkDB with Ansible

This project provides an [Ansible](http://www.ansibleworks.com/) role with
documentation and supporting scripts to help you automate the deployment of
RethinkDB. These are specific instructions for deploying a development
cluster Vagrant and VirtualBox.

The documentation and scripts are merely a starting point created to both
help familiarize you with the processes, and to bootstrap an environment
for development or evaluation. You might want to expand on them and customize
them with features specific to your needs later especially for a production
deployment.

## Vagrant Development Cluster

In some situations deploying a small cluster on your local development
machine can be quite handy. This document describes such a scenario using the
following technologies:

* [RethinkDB](http://rethinkdb.com/)
* [VirtualBox](https://www.virtualbox.org/)
* [Vagrant](http://www.vagrantup.com/) with Ansible provisioner and
  supporting *vagrant-hosts* plugin
* [Ansible](http://www.ansibleworks.com/)

Each of the virtual machines are configured with 1.5GB RAM, 2 CPU cores, and
2 network interfaces. The first interface uses NAT and has connection via the
host to the outside world. The second interface is a private network and is
used for intra-cluster communication in addition to access from the host
machine to RethinkDB's administration and API ports.

The Vagrant configuration file (`Vagrantfile`) is responsible for
configuring the virtual machines and a baseline OS installation.

The Ansible playbooks then further refine OS configuration, RethinkDB
package installation, and the initialization of the 3 nodes into a
ready-to-use cluster.

## Quick Start

Begin from the top level directory of this project and use the following
steps to get up and running:

1. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads), [Vagrant](http://downloads.vagrantup.com/), [vagrant-hosts](https://github.com/adrienthebo/vagrant-hosts), and [Ansible](http://www.ansibleworks.com/docs/intro_installation.html#latest-releases-via-pip).
2. Edit `/etc/hosts` or use included `bin/preinstall` script to add
   the following entries to your development system's `/etc/hosts` file:
 * 10.1.42.101 node1.local node1
 * 10.1.42.102 node2.local node2
 * 10.1.42.103 node3.local node3
3. `cd examples`
4. `vagrant up`
5. Access the cluster at http://node1:8080 with username **Administrator**
   and password **rethinkdb**.

This project will install Debian Jessie based cluster nodes by default. 
It can also install Ubuntu 14.04 based nodes if you prefer by changing 
the command in step 4 to the following:

```
BOX_NAME="ubuntu/trusty64" CLUSTER_HOSTS="ubuntu" vagrant up
```

*NOTE*: Due to [issue 83](https://github.com/rethinkdb/rethinkdb/issues/83)
you will see a duplicate database error when initially accessing the cluster
since the default *test* database is created on each of the 3 nodes before
the cluster is formed. Just rename/delete the databases as needed to work
around this.

If you'd like to follow a more detailed installation process with detailed
explanation of the steps and technologies, review the following sections.

## Prerequisites

Before you begin with the steps in this guide, please ensure that you have
the following prerequisites configured and installed.

### Networking

Ensure that your host machine can access the internet so that you can
download the required as needed. This includes both downloading the
prerequisite software, the VirtualBox operating system image, and RethinkDB
itself.

You'll also need to resolve the node VM hostnames to their default IP
addresses. Do this by adding the following lines to `/etc/hosts` on the 
host machine:

```
10.1.42.101 node1.local node1
10.1.42.102 node2.local node2
10.1.42.103 node3.local node3
```

This role provides a convenience script, `bin/preinstall` to automate this
step and some related steps for you.

### VirtualBox

VirtualBox is a freely available virtualization product that supports
different operating system guests running in virtual machines on Mac OS X.

Download and install the latest version from the 
[VirtualBox website](https://www.virtualbox.org/wiki/Downloads) before
proceeding with the rest of the steps in this section.

### Vagrant and Plugins

Vagrant helps automate building complete development environments on Mac OS X
with the notion of *providers* for different deployment targets like
VirtualBox and *provisioners* like Ansible for handling the actual deployment
and provisioning of the virtual machines.

This guide uses Vagrant along with a supporting plugin to automate the
creation of the VirtualBox machines which we will be using for the
RethinkDB nodes.

The plugin used in this project is [Vagrant hosts](https://github.com/adrienthebo/vagrant-hosts), which adds hostname information to `/etc/hosts`
on each virtual machine.

Visit the Vagrant [downloads page](http://downloads.vagrantup.com/) 
to get the appropriate version, and proceed with installation of the
package. After you have installed Vagrant, open a terminal and execute the
following command to install the supporting plugin:

```
vagrant plugin install vagrant-hosts
```

The command should return a success message; if the command was
successful, you can continue with the rest of the steps in this guide.

### Ansible

Install Ansible using `pip` based on the
[installation documentation](http://www.ansibleworks.com/docs/intro_installation.html#latest-releases-via-pip).

The steps are simple and involve the following:

```
sudo easy_install pip
sudo pip install ansible
```

## Bootstrap the Cluster

Now it's time for bootstrapping the cluster.

Open a terminal, change into the *examples* subdirectory of the role's
root directory and execute the top level Ansible playbook with commands like
the following:

```
cd $ROLESPATH/brianshumate.rethinkdb/examples
vagrant up
```

Yep, that's all there is to it. One command and a bit of patience while
Vagrant and Ansible work their magic. If bootstrapping a cluster for the
first time, you could see a prompt to accept the host key information 
for each node VM by ssh; be sure to answer **yes** to these prompts.

### Ubuntu

This project will install Debian Jessie based cluster nodes by default. 
It can also install Ubuntu 14.04 based nodes if you prefer by changing 
the `vagrant up` command so that it looks like this:

```
BOX_NAME="ubuntu/trusty64" CLUSTER_HOSTS="ubuntu" vagrant up
```

## Give it a Try

Once the Ansible playbooks complete without error, you can open a browser and
access the primary RethinkDB cluster node at the following URL:

```
http://node1:8080
```

*NOTE*: Due to [issue 83](https://github.com/rethinkdb/rethinkdb/issues/83)
you will see a duplicate database error when initially accessing the cluster
since the default *test* database is created on each of the 3 nodes before
the cluster is formed. Just rename/delete the databases as needed to work
around this.

### Other Operations

While the basic instructions for getting started do a fair bit of work, you
can also run the `build_cluster.yml` playbook against an already bootstrapped
collection of VMs to do the actual cluster initialization.

You can also run a subset of the tasks independently through Ansible's
tag support; here is a list of available tags by role:

**bootstrap**

* *network* : Network hostname
* *system_packages* : Install any required system packages
* *system_tuning* : Set disk scheduler specified in `defaults/main.yml`
  for the data and index volumes, and other system tuning

**rethinkdb**

* *installation* : Download and install the RethinkDB package
* *network* : RethinkDB specific firewall settings
* *service* : RethinkDB service related tasks

Here are some examples of tag usage:

Set the disk scheduler specified in `defaults/main.yml` for the data and
index volumes, and perform other system tuning:

```
ansible-playbook -i centos build_cluster.yml --tags "system_tuning"
```

Download and install the version of RethinkDB specified in `defaults/main.yml`
for the Linux distribution in use:

```
ansible-playbook -i centos build_cluster.yml --tags "installation"
```

You can also combine tags, as in the following example, which will
perform the system tuning and installation tasks:

```
ansible-playbook -i centos build_cluster.yml \
--tags "system_tuning, installation"
```

## Examples

The `examples` directory contains some basic playbooks, host inventory
examples, and Vagrant bits:

* `build_cluster.yml` installs RethinkDB and creates a cluster
* `example_hosts` example hosts inventory in format required by this project
* `Vagrantfile` example Vagrant development cluster definition
* `centos` CentOS hosts inventory for Vagrant based development cluster
* `debian` Debian hosts inventory for Vagrant based development cluster
* `redhat` Red Hat hosts inventory for Vagrant based development cluster
* `ubuntu` Ubuntu hosts inventory for Vagrant based development cluster

## Notes

1. This project functions with the following software versions:
  * Ansible version 2.0.0.2
  * VirtualBox version 5.0.4
  * Vagrant version 1.8.1
  * Vagrant Hosts version 2.4.0
2. The `bin/preinstall` shell script performs the following actions for you:
 * Adds each node's host information to the host machine's `/etc/hosts`
 * Ensures the correct permissions on Vagrant SSH private key
 * Optionally installs the Vagrant hosts plugin
3. Review the different operating system tuning changes in the following
  files to adjust or add your own:
  * `roles/bootstrap/common/templates/etc_rc.local.j2`
  * `roles/bootstrap/common/templates/iptables.j2`
  * `roles/rethinkdb/templates/iptables.j2`

## References

1. http://rethinkdb.com/
2. http://rethinkdb.com/docs/install/
