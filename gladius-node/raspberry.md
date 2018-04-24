# Setup a raspberry pi as a gladius node

## Intro
This document guides you trough the necessary steps to install the gladius node consisting of the control daemon, the edge daemon and the gladius cli on a raspberry pi.

The setup described is the `native` setup - This means we are installing the node daemons on the system without packing it into a docker container.

## Setup
### ETH Beta Wallet
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

### Install Gladius
With the the test wallet filled up, the raspbian os installed and the network configured it is time to install and setup gladius on your raspberry pi.

#### change password
Before we setup gladius login via ssh and the static ip to your raspberry pi (for windows users I recommend [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). The default username is `pi` and password is `raspberry`

After the successfull login change the password of the pi user:
```bash
passwd
```

#### update system
It is a good idea to upgrade the base system before proceeding with the gladius setup.

```bash
# update apt package definition
sudo apt-get update

# upgrade system
sudo apt-get upgrade -y
```

#### install nodejs
First we install nodejs. 
```bash
# install the node repository
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -

# install nodejs
sudo apt-get install -y nodejs
```

After the installation we need to change the default nodejs paths so we dont need superuser rights to install packages.
```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo "PATH=~/.npm-global/bin:$PATH" >> ~/.bashrc
source .bashrc
```

#### install other requirements
For the gladius installation we need make, git and other tools installed too:
```bash
sudo apt-get install -y git make g++
```

#### install gladius binaries
All gladius binaries can be installed with the node package manager
```bash
# install the cli
npm install -g gladius-cli
# install the control daemon
npm install -g gladius-control-daemon
# install the edge daemon
npm install -g gladius-edge-daemon
```

#### setup gladius services
There are no service files installed by default. To simplify running the services we need to create two systemd service units for the control end edge daemon.

```bash
# create service file for control daemon
sudo bash -c 'cat << EOF > /etc/systemd/system/gladius-control-daemon.service
[Unit]
Description=gladius control daemon
After=network.target

[Service]
Environment="PATH=/home/pi/.npm-global/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"
Type=simple
User=pi
ExecStart=/home/pi/.npm-global/bin/gladius-control
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF'

# create service file for edge daemon
sudo bash -c 'cat << EOF > /etc/systemd/system/gladius-edge-daemon.service
[Unit]
Description=gladius edge daemon
After=network.target

[Service]
Environment="PATH=/home/pi/.npm-global/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"
Type=simple
User=pi
ExecStart=/home/pi/.npm-global/bin/gladius-edge
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF'
```

With the systemd units in place we need to instruct systemd to reload the configuration
```bash
sudo systemctl daemon-reload
```

Next we instruct the services to start on system boot and start the services.
```bash
# enable services on boot
sudo systemctl enable gladius-control-daemon
sudo systemctl enable gladius-edge-daemon

# start daemons
sudo systemctl start gladius-control-daemon
sudo systemctl start gladius-edge-daemon
```

#### check gladius daemons
Lets check if both gladius daemons are running.

We can use `systemctl` to check if the services started successfully: 
```bash
# use systemctl status to check the service state
sudo systemctl status gladius-control-daemon
sudo systemctl status gladius-edge-daemon
```


With the gladius cli we can also check the status of the daemons:
```bash
# use the gladius cli to check node status
gladius-node status
```
*Attention:* The gladius node status command will complain about the missing keys. We will create them in the next chapter
 
### Configure Gladius
With the gla

## Next steps
This document is not finished. 
A few things are missing and still need to be written:

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
- https://github.com/gladiusio/gladius-wiki/blob/master/gladius-node/maintain.md
- https://github.com/gladiusio/gladius-wiki/blob/master/gladius-node/install.md
### Raspbian
- https://www.raspberrypi.org/documentation/installation/installing-images/README.md
- https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0
- https://www.w3schools.com/nodejs/nodejs_raspberrypi.asp
- https://nodesource.com/blog/running-your-node-js-app-with-systemd-part-1/
### raspberry pi
- https://github.com/RetroPie/RetroPie-Setup/wiki/Overclocking