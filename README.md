# MASS On Virtual Box Setup

## Virtual Box Setup
#### Networking: need to setup two adapters
- first adapter set to internal network
- second adapter set to bridged network

## Install Ubuntu
Install ubuntu on the vm with all default settings. Since we have our primary adapter set to the internal network, the vm 
does not have internet currently. We need to set it up
````
sudo nano /etc/network/interfaces
````
 add the following lines
 ````
auto enp0s3 
iface enp0s3 inet static
  address 192.168.1.1
  netmask 255.255.255.0

auto enp0s8 
iface enp0s8 inet dhcp
````

