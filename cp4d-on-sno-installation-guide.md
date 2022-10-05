# Installation guide Cloud Pak for Data on a Single Node Openshift cluster

## Overview

1. SNO
- 1.1 VMware
- 1.2 Connectivity
- 1.3 Create iso file
- 1.4 Load the iso file in VMware
- 1.5 Create and startup VM
- 1.6 Assisted Installer

2. NFS
- 2.1 Setup NFS service on bastion
- 2.2 Create storage class managed-nfs-storage
- 2.3 Test nfs storage class

3. CP4D
- 3.1 Setting up a client workstation
- 3.2 Setting up installation environment variables
- 3.3 Setting up projects (namespaces) on Red Hat OpenShift Container Platform
- 3.4 Creating custom security context constraints for services
- 3.5 Changing required node settings
- 3.6 Updating the global image pull secret
- 3.7 Installing the IBM Cloud Pak for Data platform and services


### 1. SNO

#### 1.1 VMware

To create a virtual environment/node for the Single Node Openshift cluster this guide is using the Gym environment of the [IBM Technology Zone](https://techzone.ibm.com/).

To request vCenter access to [vmware-gym](https://techzone.ibm.com/my/reservations/create/6241d306a81132001fcfe0d1) follow the link.

After you received the vCenter access, you will automatically get a router and a bastion.



#### 1.2 Connectivity

We could use apache guacamole to do our installation, however it is much more convenient to use your own preferred terminal and browser from your laptop.
This is why we need the ip-address of the bastion.

To get the ip-address of your bastion click on "Open your IBM Cloud environment" blue button on top of your reservation page.
This is a guacamole managed section, under "All Connections", open up "OCP Gym" to get to "SSH Gym - ... - bastion" and click on it.
Type to see the ip-address:

```
ifconfig
```

It looks something like this:

```
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> 
mtu 150 inet 192.168.252.2
```


We do need to setup a vpn for connectivity, we can do this by installing [Wireguard](https://www.wireguard.com/install/).
The config file for Wireguard can be downloaded at the bottom of the TechZone reservation page, the button is called "Download Wireguard VPN config".
Start Wireguard vpn and you have connectivity to your VMware environment including your bastion server.

You can now connect to the vpn, using the ip-address found, and the "vCenter Password":

```
ssh admin@192.168.252.2
```

And connect to VMware vSphere:

```
https://ocpgym-vc.techzone.ibm.local/ui
```

Using the "vCenter Username", looking like this: `gymuser-l5edwxqd@techzone.ibm.local`, and the same "vCenter Password".



#### 1.3 Create iso file

Go to [RedHat console](https://console.redhat.com) and login.

On the left side, click on openshift and clusters.
Click on "Create Cluster" button.
Choose tab "Datacenter".
Under Assisted Installer, click "Create cluster".

Fill out the form: (for example)
Clustername: sno-two
Base domain: example.com
Openshift version: 4.10.30
Chexk the box "Install single node OpenShift (SNO)"
Leave rest default and click Next.
Leave operators default and click Next.
Under Host discovery click on "Add host".
Leave all default, just add your public key (from your laptop).
Click generate discovery iso and download to your laptop.

Looks like:
```
4180d411-274e-48b8-be97-db00b89dd357-discovery.iso
```
It is about 106Mb large.

Note: Leave this page open, we will get back to it once the VM has started successfully.



#### 1.4 Load the iso file in VMware

We need to make the iso file available within vSphere, so we can boot a VM with it later.

log into vSphere, see paragraph 1.2 for details.
On the left side there are 4 icons, click on the database icon, then on IBMCloud/datastore-shared.
On the right side of the screen, click on "Files", then "Upload files" and upload the iso file from your laptop.



#### 1.5 Create and startup VM

On the left side of the screen, choose the second icon from the left (on the left of the database icon).
Open up the IBMCloud/ocp-gym folders and click on the gym folder inside, looks like `gym-270004pefx-l5edwxqd`.

On the right side under Actions, choose "New Virtual Machine", click Next.
Name the new VM, click Next.
Leave as is, Next.
Storage, select datastore-shared, Next.
Leave as is, Next.
Guest OS Family: `Other`.
Guest OS Version: ``Other 64-bit`, Next.
Customize hardware: 24 cores, 256Gb Ram, 120Gb disk

New CD/DVD drive: Datastore ISO File, select the uploaded iso file.
Check the Connect box as well behind it.
Click Finish.

Click on the new VM just created and start it.



#### 1.6 Assisted Installer

While the VM is booting up, it is downloading some things and connecting to RedHat.
After some minutes the host discovery page of the RedHat assisted installer will reveal a new cluster.
(Maybe there are some to-dos indicated like changing the hostname.)
Follow the steps, leaving all default, after about 30 minutes the lcuster is ready.
(View cluster events (for details) during the process.)

After the SNO cluster is ready, note down the connection info like:

```
https://console-openshift-console.apps.sno-two.example.com/
kubeadmin
xDitZ-D8I2H-vCSKi-bkIqS
```

Update your /etc/hosts file on your laptop and on the bastion, for example:

```
192.168.252.104  console-openshift-console.apps.sno-two.example.com oauth-openshift.apps.sno-two.example.com snotwo api.sno-two.example.com
```
You can now connect to Openshift, for example:

```
oc login -u kubeadmin -p xDitZ-D8I2H-vCSKi-bkIqS --server=https://api.sno-two.example.com:6443
```


### 2. NFS

#### 2.1 Setup NFS service on bastion








