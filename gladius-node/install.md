# Installing the Gladius Node

## Setup

Install [nodejs](https://nodejs.org/en/download/), then clone the
[gladius-cli](https://github.com/gladiusio/gladius-cli)
repository and install it by navigating to it's directory and running
`npm install -g`

## Install with either:

### Docker

Install [Docker](https://docs.docker.com/install/) and
[Docker Compose](https://docs.docker.com/compose/install/)

Clone the
[gladius-edge-docker](https://github.com/gladiusio/gladius-edge-docker)
repository and run the command `docker-compose up -d --build` to build and run
the containers.

### Directly

Install [pm2](http://pm2.keymetrics.io/) with `npm install pm2 -g`

Clone the
[gladius-edge-daemon](https://github.com/gladiusio/gladius-edge-daemon) and
[gladius-control-daemon](https://github.com/gladiusio/gladius-control-daemon)
repositories.

Navigate to the gladius-edge-daemon folder and run `pm2 start index.js`, then
navigate to the gladius-control-daemon folder and run the same command.

## Configuration
See [here](./maintain.md)
