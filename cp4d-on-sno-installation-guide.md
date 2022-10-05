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

```ifconfig```

It looks something like this:

```ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 150 inet 192.168.252.2```


We do need to setup a vpn for connectivity, we can do this by installing [Wireguard](https://www.wireguard.com/install/).
The config file for Wireguard can be downloaded at the bottom of the TechZone reservation page, the button is called "Download Wireguard VPN config".
Start Wireguard vpn and you have connectivity to your VMware environment including your bastion server.



#### 1.3 Create iso file



