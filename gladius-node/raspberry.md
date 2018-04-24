# Setup a raspberry pi as a gladius node

## Intro
This document guides you trough the necessary steps to install the gladius node consisting of the control daemon, the edge daemon and the gladius cli on a raspberry pi.

The setup described is the `native` setup - This means we are installing the node daemons on the system without packing it into a docker container.

## Setup
### Ethernet Beta Wallet
It is strongly recommended to use a dedicated ETH wallet just for the gladius beta tests. 
Create a new ETH wallet with the help of this [guide](https://medium.com/benebit/how-to-create-a-wallet-on-myetherwallet-and-metamask-e84da095d888).

With the new wallet in place send 1 ETH from the [ropsten test network](http://faucet.ropsten.be:3001/).

After a minute you should see 1 ETH from the ropsten testnetwork in your newly created wallet.
*Attention:* Without 1 ETH in your wallet the gladius registration process will NOT work.

### Raspberry PI
In this chapter we will install a raspberry pi with raspbian and configure the os

#### Download and Install Raspbian
- Download the current raspbian lite image from here: https://downloads.raspberrypi.org/raspbian_lite_latest
- Download etcher and install etcher: https://etcher.io/
- Insert your SD card and "burn" the raspian image to the SD card

#### Enable SSH
After the etcher tool has finished "burning" the image to the SD card reinsert the SD card into your computer.
You should see a new disk called `boot` show up.

Create an empty file called `ssh` in the boot partition to enable ssh when the system boots.

#### Boot the Raspberry PI
Insert the SD card into the PI, attach a network cable and plug the usb power cable into it.

### Network Configuration
Before we install and configure gladius on the system you need to set a static ip address and add a NAT rule (port forwarding) to your router.
Now, the problem with this step is that every router uses a different Web interface or shell for configuration. It is impossible to write a proper guide for those two steps so I recommend to google `port forwarding` and `static dhcp` together with your router model and see what you can find/

#### Static DHCP lease
For the port forwarding to work we need to fix the IP address the raspberry pi gets assigned.
In your router assign a static dhcp lease to the raspberry pi

#### Port forwarding
You need to forward TCP 8080 from the internet to your rasperry pi.
This port is used by the cdn network to retrieve cached content.

#### Restart the Raspberry PI
After you setup the static dhcp lease and the port forwarding reboot the raspberry pi.
The reboot ensures that it gets the static ip assigned.

#### change password
After the reboot login via ssh and the static ip to your raspberry pi (for windows users I recommend [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). The default username is `pi` and password is `raspberry`

After the successfull login change the password of the pi user:
```
passwd
```

### Gladius
With the the test wallet filled up, the raspbian os installed and the network configured it is time to install and setup gladius on your raspberry pi.


#### install nodejs

## Next steps
This document is not finished. 
A few things are missing and still need to be written:

- Docker based setup
- Overclocking
- Monitoring
- Log management
- Cache management
- Troubleshooting

## Weblinks
Further information for the installation process
### Gladius
- https://github.com/gladiusio/gladius-wiki/blob/master/gladius-node/maintain.md
- https://github.com/gladiusio/gladius-wiki/blob/master/gladius-node/install.md
### Raspbian
- https://www.raspberrypi.org/documentation/installation/installing-images/README.md
- https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0
- https://github.com/RetroPie/RetroPie-Setup/wiki/Overclocking