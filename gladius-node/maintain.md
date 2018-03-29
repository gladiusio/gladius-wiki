## Setup and Maintain the Gladius Node Manager

#### Gladius Control Daemon

* Run `gladius-control` to start the server
  * Expected Output:
    ```
    $ gladius-control                                                                       
    Running at http://localhost:3000
    ```
  * **Leave this running in a new window for the CLI to communicate**

#### Gladius Edge Daemon

  * Run `gladius-edge` to start the server
    * Expected Output:
      ```
      $ gladius-edge                                                                       
      Running - Use "gladius-node start" to start it
      ```
    * **Leave this running in a new window for the CLI to communicate**

#### Gladius CLI

- Set up a local static IP for the machine you will be running the Gladius node on
- Forward port 8080 on your router to that machine
- Create a [new Ethereum wallet](https://medium.com/benebit/how-to-create-a-wallet-on-myetherwallet-and-metamask-e84da095d888)
- Acquire 1 Ether on the [Ropsten testnet](http://faucet.ropsten.be:3001/) (or go [here](https://blog.bankex.org/how-to-buy-ethereum-using-metamask-ccea0703daec) if you're using Metamask)
- Run `gladius-node init` and fill out the requested
information (use the same email that you applied for the beta with)

After you execute a command it will suggest the next logical command. For example, after `init` you can run `gladius-node create` to create a new Node. As of now the Node manager only supports 1 Node per user therefore if you run `gladius-node create` multiple times you will keep overwriting your current node.

If you need help port forwarding or setting up a local ip lots of resources can be found by searching the Internet about router specific instructions

For a full list of commands, please see the [gladius-cli](https://github.com/gladiusio/gladius-cli) repo
