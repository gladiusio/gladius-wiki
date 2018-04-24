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

*Attention:* From here on the gladius control and edge daemon will always start automatically. There is *no* need to run `gladius-control` or `gladius-edge` manually in a ssh session.

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
 
### Initialise Gladius
With the gladius daemons running we can initialise the gladius node. To do this you need to run `gladius init`.
The following shows a complete `init` process.

```
gladius-node init
[Gladius-Node] What's your email? (So we can contact you about the beta):  huttersebastian@gmail.com
[Gladius-Node] What's your first name?  Sebastian
[Gladius-Node] Short bio about why you're interested in Gladius:  This is a documentation registration. Please ignore.
[Gladius-Node] Please make a new ETH wallet and paste your private key (include 0x at the beginning):
[Gladius-Node] Please enter a passphrase for your new PGP keys:
[Gladius-Node] User profile created! You may create a node with gladius-node create
[Gladius-Node] If you'd like to change your information run gladius-node init again
```

*Attention:* 
- When you enter your wallets private key - including `0x` at the beginning - the shell will not show any entered characters. The same is true when you enter a password. Entered characters will *not* shown! 
- Please make sure you store the password for your GPG keys safely. You need the password for the next steps


#### Create the gladius node
After successfull initialization you can create the node. The following shows a complete `create` process.

```
gladius-node create
[Gladius-Node] Please enter the passphrase for your PGP private key:
[Gladius-Node] Creating Node contract, please wait for tx to complete (this might take a couple of minutes)
[Gladius-Node] Transaction: 0x604ef8812c7d719f2fa31018855edc28a480eb966488db56fbc4390974e15924  [Success]
[Gladius-Node] Setting Node data, please wait for tx to complete (this might take a couple of minutes)
[Gladius-Node] Transaction: 0x32392f13ee168ceb277574be3f12f694397e5a884438641c8b2dcc9a7e31a71b  [Success]
[Gladius-Node] Node successfully created and ready to use
[Gladius-Node] Use gladius-node apply to apply to a pool
```

*Attention:* 
- If the process hangs while creating the node contract make sure you have 1 ETH from the [ropsten test network](http://faucet.ropsten.be:3001/) in the specified wallet. If not stop the process (Ctrl+C) and transfer 1 ETH. After the ETH has arrived re-run `create`.
- As with the `init` process the password for your GPG keys is *not* shown when you enter it.

#### Apply for the beta pool
After successfull node creation you can apply for the beta pool. You need the beta pool address - you can find it in the beta invitation email.

The following shows a complete `apply` process:
```
gladius-node apply
[Gladius-Node] Please enter the passphrase for your PGP private key:
[Gladius-Node] Please enter the address of the pool you want to join:   0xABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCD
[Gladius-Node] Transaction: 0xa8140815877b76c9c6409f4a82a88a5c260b67b0468eceb6461b14d7bbb0f1e7  [Success]
[Gladius-Node] Application sent to Pool!
[Gladius-Node] Use gladius-node check to check your application status
```

*Attention:* 
- The pool address in the example is *not* a valid pool address. please check your beta invitation email for the correct address
- As with the `init` process the password for your GPG keys is *not* shown when you enter it.

#### Check your pool application
To check your pool application status you can run `check`
```
gladius-node check
[Gladius-Node] Please enter the address of the pool you want to check on:   0xABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCD
[Gladius-Node] Pool: 0xABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDsusch  [Application Status: Pending]
[Gladius-Node] Wait until the pool manager accepts your application in order to become an edge node
```

## Use Gladius

#### start gladius node
When your application is approved you can start your gladius node
```
gladius-node start
[Gladius-Node] Gladius Edge Daemon running, you are now an edge node! (If you're a part of a pool)
```

#### check the gladius node
```
gladius-node status
[Gladius-Node] Gladius Control Daemon server is running!
[Gladius-Node] Gladius Edge Daemon running, you are an edge node!
[Gladius-Node] If you'd like to stop, run gladius-node stop
```

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