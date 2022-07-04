# Bridge Multipass _(Ubuntu server 20.04LTS)_
### Introduction
---
Multipass is a lightweight VM manager for Linux, Windows and macOS. It's designed for developers who want a fresh Ubuntu environment with a single command. Now this documents will help you config multipass create instance have network bridge with server.
### Network bridge for multipass
---
A network bridge is a computer networking device that creates a single, aggregate network from multiple communication networks or network segments. This function is called network bridging.[1] Bridging is distinct from routing. Routing allows multiple networks to communicate independently and yet remain separate, whereas bridging connects two separate networks as if they were a single network.

![hub-in-networking](https://user-images.githubusercontent.com/62330937/177121273-c58daa1e-d9d1-437d-94bd-5bb254800e56.jpeg)

1. Any LXD will have ip same to the server.
Example: Router (192.168.1.1) -> server (192.168.1.3) and any lxd have create will have 192.168.1.x

### Install multipass.
---
To install Multipass, simply execute:

```
sudo snap install multipass
```
For architectures other than amd64, you’ll need the beta channel at the moment.
You can also use the edge channel to get the latest development build:
```
sudo snap install multipass --edge
```
When install multipass success, let's go to config on server
### Config on server
---
`***Require` 
Install package require on ubuntu server
```bash
sudo apt install network-manager
```
NetworkManager will manage all networks on server, minitoring status networks. Beside you can manage diffirent networks by deep manager (create, mod, delete). 
Then we need edit netplan for static ip makesure for ssh-server .
```sh
cd /etc/netplan/ # Move to folder netplan
sudo nano *.installer-config.yml # File yml in netplan
```
Then we will config file like bellow:
```python
network:
  version: 2
  renderer: NetworkManager # Make sure install package network-manager first
  ethernets:
    enp4s0:
      optional: true # Ignore waiting network
      dhcp4: no
      addresses: [192.168.1.37/24] # Static IPv4
      gateway4: 192.168.1.1 # Set gateway from router maybe 192.168.0.1
      nameservers:
          addresses: [8.8.8.8, 192.168.1.1]
    enp5s0:
      optional: true
      dhcp4: true
```
After config like on top need restart netplan
```
sudo netplan generate
sudo netplan apply
```
Check Network manager work!
```sh
nmcli con show 
# or 
nmcli connection show
```
Output like this is successful:
```
Name             Type      Description
docker0          bridge    Network bridge
docker_gwbridge  bridge    Network bridge
enp1s0           ethernet  Ethernet device
mpbr0            bridge    Network bridge for Multipass
```
- The `enp1s0` is ethernet we will use it to bridge network.
- The `docker0` is Network bridge.
- The `docker_gwbridge` is Network bridge.
### Let's create instance multipass
---

To show list instance multipass:
```sh
multipass list
```
Config varible local multipass first we need to update local variable for multipass can understand the run time on ubuntu (have tested in ubuntu server 20.04LTS). First will will check backend use what's a `driver` multipass. This mistake it will show in log is `Network error message: Feature is not implemented on this backend`
```sh
multipass get local.driver
```
If `local.driver` have response is not `lxd`, need to change to lxd bellow command:
```sh
multipass set local.driver=lxd
```
According on top we can use `multipass networks` to check networks on server and choose right name networks to set bridged. 
```sh
multipass set local.bridged-network = `Name network`
```
Let create new instance multipass using network bridge.
```
multipass launch -n something -c 4 -m 2gb --bridged
```
Description for this command:
- `-n or --name`: Set name instance
- `-network=bridge,mode=manual or --bridged`: Auto setup bridge for instance
- `-c or -cpu`: Set number core for this instance
- `-m or -memory`: Provide amount memory for this instance mb, gb


