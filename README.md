# Bridge Multipass _(Ubuntu server 20.04LTS)_
### Introduction
---
Multipass is a lightweight VM manager for Linux, Windows and macOS. It's designed for developers who want a fresh Ubuntu environment with a single command. Now this documents will help you config multipass create instance have network bridge with server.
### Install multipass
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
Let create new instance multipass using network bridge.
```
multipass launch -n something -c 4 -m 2gb --bridged
```
Description for this command:
- `-n or --name`: Set name instance
- `-network=bridge,mode=manual or --bridged`: Auto setup bridge for instance
- `-c or -cpu`: Set number core for this instance
- `-m or -memory`: Provide amount memory for this instance mb, gb


