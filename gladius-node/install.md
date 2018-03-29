# Installing the Gladius Node Manager

## Dependencies (direct install only)

### Node.js

Node.js provides a general installation guide [here](https://nodejs.org/en/download/package-manager/) but we will walk through the installation for Windows, Ubuntu, and macOS.

We based this application off of the latest branch (9.9.0) at the time of this writing.

Here are some shortcuts to commands

* Windows
  * Download Installer, [here](https://nodejs.org/en/#download)
  * Select the latest, 9.9.0+
* Ubuntu
  * `curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -`
  * `sudo apt-get install -y nodejs`
  * Change Global Installation Directory
    * Our packages requires some dependencies that require superuser access if installed in the default Ubuntu paths. We recommend changing the default installation of global node modules to `~/.npm-global` as stated in the [npm.js docs](https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-two-change-npms-default-directory). We included the commands below:
      * Run `mkdir ~/.npm-global`
      * Run `npm config set prefix '~/.npm-global'`
      * Add `export PATH=~/.npm-global/bin:$PATH` to your `.profile` of `.zshrc` file
      * Run `source ~/.profile`

    * Another option is to use [NVM](https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-one-reinstall-with-a-node-version-manager) to handle permissions.
* macOS
  * Install Homebrew, [instructions](https://brew.sh/)
    * `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
    * `brew install node`

### Git

* Windows
  * https://gitforwindows.org
* Ubuntu
  * `apt-get install git`
* macOS
  * Comes default with mac but can also be installed via [Homebrew](https://brew.sh/) (`brew install git`)

### Python 2.7
* Windows
  * Run `npm install --global --production windows-build-tools` in cmd.exe as Administrator
  * Run `setx PYTHON "%USERPROFILE%\.windows-build-tools\python27\python.exe"` in cmd.exe
  * Restart cmd.exe
* Ubuntu/macOs
  * Comes default but can also be installed through their [website](https://www.python.org/)

## Gladius Software

### Using Docker

Install [Docker](https://docs.docker.com/install/) and
[Docker Compose](https://docs.docker.com/compose/install/)

Clone the
[gladius-edge-docker](https://github.com/gladiusio/gladius-edge-docker)
repository and run the command `docker-compose up -d --build` to build and run
the containers. After you can just run `docker-compose up -d`

### Direct install

#### Gladius CLI

  * Run `npm install -g gladius-cli`

#### Gladius Control Daemon

* Run `npm install -g gladius-control-daemon`

#### Gladius Edge Daemon

  * Run `npm install -g gladius-edge-daemon`
