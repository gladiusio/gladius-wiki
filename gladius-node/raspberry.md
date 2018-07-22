# Setup a raspberry pi as a gladius node

## Intro
This document guides you trough the necessary steps to install the gladius node consisting of the control daemon, the edge daemon and the gladius cli on a raspberry pi.

The setup described is the `native` setup - This means we are installing the node daemons on the system without packing it into a docker container.

## Setup the Raspberry Pi

In this chapter we will install a raspberry pi with raspbian and configure the os

### Download and Install Raspbian
- Download the current raspbian lite image from here: https://downloads.raspberrypi.org/raspbian_lite_latest
- Download etcher and install etcher: https://etcher.io/
- Insert your SD card and "burn" the raspian image to the SD card

### Enable SSH
After the etcher tool has finished "burning" the image to the SD card reinsert the SD card into your computer.
You should see a new disk called `boot` show up.

Create an empty file called `ssh` in the boot partition to enable ssh when the system boots.

### Boot the Raspberry PI
Insert the SD card into the PI, attach a network cable and plug the usb power cable into it.

## Network Configuration
Before we install and configure gladius on the system you need to set a static ip address and add a NAT rule (port forwarding) to your router.

Please have a look at our [Port Forward Guides](../router.md). The list is not complete. If you can not find your router model a google search with your router model and the term 'port forwarding' should help you out.

### Static DHCP lease
For the port forwarding to work we need to fix the IP address the raspberry pi gets assigned.
In your router assign a static dhcp lease to the raspberry pi

### Port forwarding
You need to forward TCP 8080 from the internet to your rasperry pi.
This port is used by the cdn network to retrieve cached content.

### Restart the Raspberry PI
After you setup the static dhcp lease and the port forwarding reboot the raspberry pi.
The reboot ensures that it gets the static ip assigned.


## Install Gladius on the Raspberry Pi
With the raspbian os installed and the network configured, it is time to install and setup gladius on your raspberry pi.

### change password
Before we setup gladius login via ssh and the static ip to your raspberry pi (for windows users I recommend [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). The default username is `pi` and password is `raspberry`

After the successfull login change the password of the pi user:
```bash
passwd
```

### update system
It is a good idea to upgrade the base system before proceeding with the gladius setup.

```bash
# update apt package definition
sudo apt-get update

# upgrade system
sudo apt-get upgrade -y
```

### Install gladius
The required gladius binaries can be installed with the help of a small installer script.

First, we install the gladius binaries then we start the services and make sure they automatically start when the pi is rebooted.
```bash
# download and execute the installer script
curl -s https://raw.githubusercontent.com/gladiusio/gladius-node/master/installers/install.sh | sudo bash

# install networkd and controld as systemd services
sudo gladius-controld install
sudo gladius-networkd install

# reload the systemd daemon
sudo systemctl daemon-reload

# make sure the two daemons are started automatically during boot
sudo systemctl enable GladiusControlDaemon
sudo systemctl enable GladiusNetworkDaemon

# start the services
sudo systemctl start GladiusControlDaemon
sudo systemctl start GladiusNetworkDaemon
```

Lets check if the two gladius services are running

```bash
# check if controld is up and running
systemctl status GladiusControlDaemon

● GladiusControlDaemon.service - Gladius Control Daemon
   Loaded: loaded (/etc/systemd/system/GladiusControlDaemon.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2018-07-21 19:40:28 UTC; 19s ago
 Main PID: 11590 (gladius-control)
   CGroup: /system.slice/GladiusControlDaemon.service
           └─11590 /usr/local/bin/gladius-controld /root/.config/gladius

Jul 21 19:40:28 raspberrypi systemd[1]: Started Gladius Control Daemon.
Jul 21 19:40:30 raspberrypi gladius-controld[11590]: Starting API at http://localhost:3001


# check if netwodk is up and running
systemctl status GladiusNetworkDaemon

● GladiusNetworkDaemon.service - Gladius Network (Edge) Daemon
   Loaded: loaded (/etc/systemd/system/GladiusNetworkDaemon.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2018-07-21 19:40:31 UTC; 1min 16s ago
 Main PID: 11608 (gladius-network)
   CGroup: /system.slice/GladiusNetworkDaemon.service
           └─11608 /usr/local/bin/gladius-networkd /root/.config/gladius

Jul 21 19:40:31 raspberrypi systemd[1]: Started Gladius Network (Edge) Daemon.
Jul 21 19:40:31 raspberrypi gladius-networkd[11608]: Loading config
Jul 21 19:40:31 raspberrypi gladius-networkd[11608]: Starting...
Jul 21 19:40:31 raspberrypi gladius-networkd[11608]: 2018/07/21 19:40:31 Loading website: demo.gladius.io
Jul 21 19:40:31 raspberrypi gladius-networkd[11608]: 2018/07/21 19:40:31 Loaded route: /.html
Jul 21 19:40:31 raspberrypi gladius-networkd[11608]: 2018/07/21 19:40:31 Loaded route: /anotherroute.html
Jul 21 19:40:31 raspberrypi gladius-networkd[11608]: Started RPC server and HTTP server.
```

## Configure the gladius-node
With both services running it is time now to create a wallet and apply for a pool.

First we create a new wallet and then send test ether from the ropsten test network to the wallet.
Afterwards, we deploy the smart contract and send an approval request to the pool manager.

You can find a more detailed overview over the nexessary steps in the [Gladius Node Readme](https://github.com/gladiusio/gladius-node#gladius-cli).

```bash
# the first run of `create` creates a new wallet.
# please write down your password - you will need it later
gladius create
[Gladius] Create a passphrase for your new wallet:  ********
[Gladius] Confirm your passphrase:  ********

Wallet Address: 0x6871B6CFAf856eaB3d2B838f588D70Ac5fEd1804

Please add test ether to your new wallet from a ropsten faucet

Run gladius create again after you've acquired your test ether
```

With the wallet address visit the [ropsten faucet](http://faucet.ropsten.be:3001/) and send 1 eth to your wallet.
You can check if the eth has arrived with the [ropsten etherscan site](https://ropsten.etherscan.io/).

```bash
# with the test eth in place re-run the create step to setup the gladius-node
gladius create
[Gladius] What is your name? Sebastian Hutter
[Gladius] What is your email? mail@sebastian-hutter.ch
[Gladius] Please type your wallet passphrase:  ********

Tx: 0x468fb6bfe804de8356ed0df25f823e50465aca3f98d6538c8adf05a02dd37f98   Status: Pending...
Tx: 0x468fb6bfe804de8356ed0df25f823e50465aca3f98d6538c8adf05a02dd37f98   Status: Successful
Node created!

Tx: 0x7d6958813a51fd94556deea49916484a5767f6b87540b0ec5afc283c8769c07b   Status: Pending..
Tx: 0x7d6958813a51fd94556deea49916484a5767f6b87540b0ec5afc283c8769c07b   Status: Successful
Node data set!

Node Address: 0x94cc1ed72773d3611632e83dffc2231dd6847fbd

Use gladius apply to apply to a pool
```

The next step is to apply to a gladius pool.
*Attention:* There are no official gladius pools just yet.

```bash
gladius apply
[Gladius] Pool Address:  0x123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ12345
[Gladius] Please type your wallet passphrase:  ********

Tx: 0x53a47e2188485ed9b7ce52c412420d0527a5915f66e98cd00a5de678c9259835   Status: Pending...
Tx: 0x53a47e2188485ed9b7ce52c412420d0527a5915f66e98cd00a5de678c9259835   Status: Successful

Application sent to pool!

Use gladius check to check the status of your application
```

The approval process can take from a few minutes to a few days. Depending how the gladius pool admin is approving the requests.

You can run the `check` command to check the status of the request
```bash
[Gladius] Pool Address:  0x123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ12345
Pool: 0x123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ12345         Status: Pending

Use gladius node start to start the node networking software
```

## Use Gladius

### start gladius node
You can start the gladius node anytime. Your request doesn't need to be approved for this step but as long as your node is not approved it will not serve any content.
```bash
gladius node start
Network Daemon:  Started the server

Use gladius node stop to stop the node networking software
Use gladius node status to check the status of the node networking software
```

### check the gladius node
Check if the gladius node network daemon is running and able to serve content.
```bash
gladius node status
Network Daemon:  Server is running

Use gladius node start to start the node networking software
Use gladius node stop to stop the node networking software
```

### check the gladius node profile
Print the gladius node profile data
```bash
gladius profile

Account Address: 0x6871B6CFAf856eaB3d2B838f588D70Ac5fEd1804
Node Address: 0x94cc1ed72773d3611632e83dffc2231dd6847fbd
Node Name: Sebastian Hutter
Node Email: mail@sebastian-hutter.ch
Node IP: 212.51.159.27
```

## Next steps
This document is not finished. 
A few things are missing and still need to be written:

- Change service user from root to non-privileged account
- Docker based setup
- Overclocking
- Monitoring
- Log management
- Cache management
- Troubleshooting
- Backup of configuration and keys

## Weblinks
Further information for the installation process
### Gladius
- https://github.com/gladiusio/gladius-node/blob/master/README.md
### Raspbian
- https://www.raspberrypi.org/documentation/installation/installing-images/README.md
- https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0
### raspberry pi
- https://github.com/RetroPie/RetroPie-Setup/wiki/Overclocking