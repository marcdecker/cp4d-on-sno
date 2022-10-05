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

Update your /etc/hosts file on your laptop and on the bastion (as root user), for example:

```
192.168.252.104  console-openshift-console.apps.sno-two.example.com oauth-openshift.apps.sno-two.example.com snotwo api.sno-two.example.com
```
You can now connect to Openshift, for example:

```
oc login -u kubeadmin -p xDitZ-D8I2H-vCSKi-bkIqS --server=https://api.sno-two.example.com:6443
```


### 2. NFS

#### 2.1 Setup NFS service on bastion

We need to organize some file storage for Openshift.
We can simply setup an NFS service on the bastion.

Be root on the bastion (you probably logged in as user admin).

```
sudo -i
```

Make directory for nfs and open it up:

```
mkdir /nfs
chmod 777 /nfs
```

Install nfs-utils (usually already present) and start nfs service:

```
yum -y install nfs-utils

systemctl enable --now nfs-server rpcbind

systemctl status nfs-server
```

Update nfs config:

```
vi /etc/exports

add line:

/nfs *(rw,sync)

exportfs -rav
```

Open up the firewall for nfs:

```
sudo firewall-cmd --add-service=nfs --permanent
sudo firewall-cmd --add-service={nfs3,mountd,rpc-bind} --permanent 
sudo firewall-cmd --reload 
```

Enable SELinux boolean:

```
setsebool -P nfs_export_all_rw 1
```



#### 2.2 Create storage class managed-nfs-storage

Login to openshift, see for example paragrapgh 1.6.

Follow the steps outlined [here](https://www.ibm.com/support/pages/how-do-i-create-storage-class-nfs-dynamic-storage-provisioning-openshift-environment).

Note for step 5:

```
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: ip-address-of-bastion
            - name: NFS_PATH
              value: /nfs
      volumes:
        - name: nfs-client-root
          nfs:
            server: ip-address-of-bastion
            path: /nfs
```

Put the ip-address of the bastion node here twice, and the nfs path in twice!

After step 8 the storage class is ready, you can do the tests outlined if you wish.

```
[root@bastion-gym-lan ~]# oc get sc
NAME                  PROVISIONER                                   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
managed-nfs-storage   k8s-sigs.io/nfs-subdir-external-provisioner   Delete          Immediate           false                  23h
```



### 3. CP4D

We are doing an express installation, following the [IBM knowledge center](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.5.x?topic=installing) documents. Assuming version 4.5.2.

`I want to ensure that a specific version of the software is installed on my cluster.`



#### 3.1 Setting up a client workstation

We need an oc client, and is was already installed on the bastion.

Installing the IBM Cloud Pak for Data command-line interface:

```
wget https://github.com/IBM/cpd-cli/releases/download/v11.2.0/cpd-cli-linux-EE-11.2.0.tgz -P /tmp

tar -xf /tmp/cpd-cli-linux-EE-11.2.0.tgz -C /usr/local/bin/

mv -f /usr/local/bin/cpd-cli-linux-EE-11.2.0-40/* /usr/local/bin

cpd-cli version
```

Make sure cpd-cli is installed and running:

```
cpd-cli manage
```



#### 3.2 Setting up installation environment variables

Example:

```
vi cpd_vars.sh


#===============================================================================
# Cloud Pak for Data installation variables
#===============================================================================

# ------------------------------------------------------------------------------
# Cluster
# ------------------------------------------------------------------------------

export OCP_URL=https://api.sno-two.example.com:6443
export OPENSHIFT_TYPE=self-managed
export OCP_USERNAME=kubeadmin
export OCP_PASSWORD=xDitZ-D8I2H-vCSKi-bkIqS
# export OCP_TOKEN=<enter your token>


# ------------------------------------------------------------------------------
# Projects
# ------------------------------------------------------------------------------

export PROJECT_CPFS_OPS=ibm-common-services        
export PROJECT_CPD_OPS=ibm-common-services
export PROJECT_CATSRC=openshift-marketplace
export PROJECT_CPD_INSTANCE=cpd-instance
# export PROJECT_TETHERED=<enter the tethered project>


# ------------------------------------------------------------------------------
# Storage
# ------------------------------------------------------------------------------

export STG_CLASS_BLOCK=managed-nfs-storage
export STG_CLASS_FILE=managed-nfs-storage

# ------------------------------------------------------------------------------
# IBM Entitled Registry
# ------------------------------------------------------------------------------

export IBM_ENTITLEMENT_KEY=get your own key [here](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.5.x?topic=information-obtaining-your-entitlement-api-key).


# ------------------------------------------------------------------------------
# Private container registry
# ------------------------------------------------------------------------------
# Set the following variables if you mirror images to a private container registry.
#
# To export these variables, you must uncomment each command in this section.

# export PRIVATE_REGISTRY_LOCATION=<enter the location of your private container registry>
# export PRIVATE_REGISTRY_PUSH_USER=<enter the username of a user that can push to the registry>
# export PRIVATE_REGISTRY_PUSH_PASSWORD=<enter the password of the user that can push to the registry>
# export PRIVATE_REGISTRY_PULL_USER=<enter the username of a user that can pull from the registry>
# export PRIVATE_REGISTRY_PULL_PASSWORD=<enter the password of the user that can pull from the registry>


# ------------------------------------------------------------------------------
# Cloud Pak for Data version
# ------------------------------------------------------------------------------

export VERSION=4.5.2


# ------------------------------------------------------------------------------
# Components
# ------------------------------------------------------------------------------
# Set the following variable if you want to install or upgrade multiple components at the same time.
#
# To export the variable, you must uncomment the command.

# export COMPONENTS=cpfs,scheduler,cpd_platform,<component-ID>
export COMPONENTS=cpfs,cpd_platform


chmod 700 cpd_vars.sh

source ./cpd_vars.sh
```



#### 3.3 Setting up projects (namespaces) on Red Hat OpenShift Container Platform

We need two projects in Openshift:

```
echo $PROJECT_CPFS_OPS
echo $PROJECT_CPD_INSTANCE

oc new-project ${PROJECT_CPFS_OPS}
oc new-project ${PROJECT_CPD_INSTANCE}
```



#### 3.4 Creating custom security context constraints for services

```
cpd-cli manage login-to-ocp --username=${OCP_USERNAME} --password=${OCP_PASSWORD} --server=${OCP_URL}

cpd-cli manage apply-scc \
--cpd_instance_ns=${PROJECT_CPD_INSTANCE} \
--components=wkc

verify:

oc adm policy who-can use scc wkc-iis-scc \
--namespace ${PROJECT_CPD_INSTANCE} | grep "wkc-iis-sa"
```



#### 3.5 Changing required node settings

CRI-O container settings:

```
cpd-cli manage login-to-ocp \
--username=${OCP_USERNAME} \
--password=${OCP_PASSWORD} \
--server=${OCP_URL}


cpd-cli manage apply-crio \
  --openshift-type=${OPENSHIFT_TYPE}
```

Kernel parameter settings:

```
cpd-cli manage apply-db2-kubelet \
--openshift-type=${OPENSHIFT_TYPE}
```



#### 3.6 Updating the global image pull secret

```
cpd-cli manage add-icr-cred-to-global-pull-secret \
${IBM_ENTITLEMENT_KEY}

watch oc get nodes
```

Verify:

```
oc get secret/pull-secret -n openshift-config -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d | jq
```



#### 3.7 Installing the IBM Cloud Pak for Data platform and services

Express installations.

Creating OLM objects for an express installation.

```
echo $VERSION
echo $COMPONENTS

cpd-cli manage apply-olm \
--release=${VERSION} \
--components=${COMPONENTS}
```

```
oc patch NamespaceScope common-service \
-n ${PROJECT_CPFS_OPS} \
--type=merge \
--patch='{"spec": {"csvInjector": {"enable": true} } }'

verify:

cpd-cli manage get-olm-artifacts \
--subscription_ns=${PROJECT_CPFS_OPS}
```

Installing components in an express installation

Note: When you use NFS storage, both ${STG_CLASS_BLOCK} and ${STG_CLASS_FILE} point to the same storage class, typically managed-nfs-storage.

```
echo $COMPONENTS
echo $VERSION
echo $PROJECT_CPD_INSTANCE
echo $STG_CLASS_BLOCK
echo $STG_CLASS_FILE

cpd-cli manage apply-cr \
--components=${COMPONENTS} \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INSTANCE} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true


cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INSTANCE}
```

To get the initial admin password after successfull installation:

```
oc extract secret/admin-user-details --keys=initial_admin_password --to=-
```

Get the route to Cloud Pak for Data, needed to get the url for your browser:

```
oc get route -n cpd-instance

NAME   HOST/PORT                                   PATH   SERVICES        PORT                   TERMINATION            WILDCARD
cpd    cpd-cpd-instance.apps.sno-two.example.com          ibm-nginx-svc   ibm-nginx-https-port   passthrough/Redirect   None
```

