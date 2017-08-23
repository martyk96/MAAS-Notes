# MAAS On Virtual Box Setup

## Table of Contents
- [Virtual Box Setup](#virtual-box-setup)
- [Install Ubuntu](#install-ubuntu)
- [Install MAAS Server](#install-maas-server)
- [Setup WebUI](#setup-webui)
- [Setup DHCP Server](#setup-dhcp-server)
- [Setup Nodes](#setup-nodes)
  - [Virtual Box](#virtual-box)
  - [WebUI](#webui)
    - [Commissioning](#commissioning)
    - [Deployment](#deployment)
    - [Getting into the Machine](#getting-into-the-machine)
- [SSH Keys](#ssh-keys)

## Virtual Box Setup
#### Memory
I used 4Gb but that's probably overkill
#### Storage
I used 20Gb but once again, probably overkill
#### Networking
- enable 2 network adapters
- first network adapter attach to bridge
- second network adapter attach to internal network

leave all other vm settings the same

## Install Ubuntu
Install ubuntu server on the vm with all default settings.

Once ubuntu is up and running, the internal network will need to be configired: 
````
sudo nano /etc/network/interfaces
````
 add the following lines
````
auto enp0s8 
iface enp0s8 inet static
  address 192.168.1.1
  netmask 255.255.255.0
````
Then bring the network up:
````
sudo ifup enp0s8
````

Now all of the networking interfaces are set up. We are ready to install MAAS

## Install MAAS Server
With a fresh install of Ubuntu, an update will need to be performed in order to get the latest version of MAAS
````
sudo apt-get update
sudo apt-get upgrade
````
and now install maas
````
sudo apt-get install maas
````

## Setup WebUI
With maas installed run the following to set up the webUI
````
sudo maas createadmin
````
provide a username, password and email. We will add the ssh keys later so skip that for now

The webUI is now up, to get to it navigate to http://$API_HOST:5240/MAAS where $API_HOST is the hostname or IP address of the region API server

To find this address
````
ifconfig
````
and use the address associated with enp0s3


When first logging in, the webserver will prompt you with settings. Keep everything default on the first page and continue at the bottom. The second page will prompt you for ssh keys. See [SSH Keys](#ssh-keys) on how to set this up properly

The webUI has now been setup

Now would be a good time to reboot the server

## Setup DHCP Server
Now that the webUI is all setup, we need to setup the DHCP server to install our nodes. Navigate to the 'Subnet' tab at the top and then click on the 'VLAN' associated with the 192.168.1.0 network. Once there, click the 'Take Action' drop down menu and select 'Provide DHCP'. Leave all deafults and enable. The DHCP server is now set. To enusure that the server is on the correct network enter the following command:
````
sudo dpkg-reconfigure maas-region-controller
````
and set the address to '192.168.1.1'

reboot

## Setup Nodes
#### Virtual Box
When creating a new client, certain setting will need to be setup for on virtual box. Here we will demonstrate a setup for Ubuntu Server install

Create a new vm and choose the OS as Linux and Ubuntu 64bit, leave memory and storage as is, the default should be plenty. Once the vm has been created, got to the settings and click in the 'System' tab. Here, check of 'Network' under boot order and move it to the top. This will make our machines go into a pxe boot. Now, navigate to the 'Network' tab and enable network adapter 1 and attach it to 'internal network'. Click 'OK' to save changes. 

Start the machine, the first time you will be prompted to select a disk to boot off of, click 'Cancel'. We want to boot off of the network.

If everything works, the vm should shut down on its own, and a new node should appear on the webUI.

#### WebUI
After a new node has been discovered, it will need to be commissioned and then deployed. But before that, a powertype needs to be selected for the node. 
- click on the node and then navigate to the 'Power' tab
- select manual under the 'Power Type' dropdown
- save changes

(The powertype allow the control over the power on the machine via the maas server. At this time, I do not know the powertype for virtual box vm's so we will do this manually).

We can now commission and deploy the node

###### Commissioning
To commission a node, simply selct the node you wish to commission and choose commission under the 'Take Action' dropdown in the top right corner.

Once the machine starts commissioning, we need to manually start the vm. If we had the proper powertype set up for the VM, it would start up on its own.

When the staus of the node changes to 'Ready' on the webUI, it has finished commissioning and the VM needs to be shut down.

We now move onto Deployment

###### Deployment
The next logical step after commissioning a node is to deploy it. To deploy a ready node, simply select the node you wish to deploy, and select 'Deploy' from the 'Take Action' dropdown bar. Click 'Deploy Machine' to start deployment.

Once again, we will need to manually start the vm.

When the status of the node changes to the name of the OS we installed (Ubuntu xx.x), the node has been successfuly deployed. Congratulations! But we are not done yet

###### Getting into the Machine
When the node deployed, it is given a username ('ubuntu') but no password to log in with. This is a problem and will not allow any login on the physical machine itself. We will need to ssh into the machine with the server that has the generated SSH key that we set up earlier. 

In order to achineve this, the ssh agent will need to be running and we need to have the private key on the account that we are using to ssh, and the public key on the machine that we are ssh'ing into. See [SSH Keys](#ssh-keys)
````
eval $(ssh-agent -s)
sudo ssh-add
````
now that the agent is up and running, simply ssh into the machine. Once in the machine, you can create a password for the ubuntu account, or you can create your own. Up to you!

(Note: to find the IP address of your new node, navigate to the 'DNS' tab, click on the maas domain, and a list of all nodes and their IP's will be shown)

## SSH Keys
Once a node has been deployed, the only user account on the machine is 'ubuntu' with no password, so it is impossible to login to the machine directly. We will need to ssh into the account and then create a password. To do this, a private and public key will be made on the maas server and the public key will be added to the webUI. This will need to be done before a node is deployed since the public key from the maas server is copied over to the node during deployment.

Create folder for ssh keys
````
mkdir ~/.ssh
chmod 700 ~/.ssh
````
Create a new key:
````
sudo ssh-keygen -t rsa
````
and place in your home directory with any name

Once that is done, we will need to copy the public key to the webUI. Easiest way to do that is to SSH into the webserver, navigate to the file and copy the key then paste on server webUI
