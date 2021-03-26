# Deploying your custom smart contracts on Celo with Truffle
Let’s say you have a smart contract you’ve written up — something world changing and exciting (or maybe it's a simple hello world). How would you go about deploying it on the Celo network?  

Read on to learn how to deploy smart contracts with Celo on Truffle. 

# Prerequisites

This tutorial assumes you have a smart contract which you've written that you'd like to deploy on Celo. 

First things first. We'll be using Truffle to deploy your contract, so make sure you have truffle installed. If you don't, run the following line of code in your terminal: 

`npm install -g truffle`

# Project setup 

First, open the terminal and make a new project folder. We’ll call if celoSmartContract:

`mkdir celoSmartContract && cd celoSmartContract`

Next, let’s initialize the project directory with NPM

`npm init -y`

After it’s been initialized, we’ll need to install some additional packages. Here’s an overview of them:
* ContractKit is a package created by the Celo team to aid in Celo development
* Dotenv is used for reading environment variables in our code
* Web3 is a library which facilitates our interactions with the blockchain

Install all of the above using:

`npm install -—save @celo/contractkit dotenv web3`

After all the NPM packages have installed, run this command to initialize Truffle: 

``truffle init``

Here's what a successful run of truffle init will look like: 

![truffle init](https://i.imgur.com/JF6zdoT.png)

# Contract setup

This tutorial is primarily concerned with deploying an existing smart contract to Celo. For that reason, we won't be going over Solidity here. 

After you've initialized truffle and NPM into your project, your project structure should now look like this: 

![project structure](https://i.imgur.com/JpTqWLJ.png)

## The Contracts folder

For this tutorial, we will deploy the **Migrations.sol** contract that is initialized with Truffle. 

Here's what it looks like: 

```// SPDX-License-Identifier: MIT
pragma  solidity >=0.4.22 <0.9.0;

contract  Migrations {

  address  public owner =  msg.sender;

  uint  public last_completed_migration;
    modifier  restricted() {
      require(msg.sender == owner,"This function is restricted to the contract's owner");
      _;
    }
    
  function  setCompleted(uint completed) public restricted {
    last_completed_migration = completed;
  }
}
```

**Note**: If you want to deploy a smart contract you've written, delete the **Migrations.sol** file in the **contracts/** folder and make a new file containing your smart contract. 

## The migrations folder

Files in the **migrations/** folder are used as deployment scripts. For each Solidity file you want to deploy, you'll need a corresponding deployment script. 

For the default Migrations.sol contract, the migration file looks like this: 

```
const  Migrations  = artifacts.require("Migrations");

module.exports  =  function  (deployer)  {
  deployer.deploy(Migrations);
};
```

**Note:** If you want to deploy your custom Solidity contract, replace the Migrations variable and contents with your contract's file name: 

```
const YOUR_CONTRACT_NAME = artifacts.require("YOUR_CONTRACT_NAME"); 

module.exports  =  function  (deployer)  {
	deployer.deploy(YOUR_CONTRACT_NAME);
};
```


# Connecting to a Testnet node

Let's create a .env file in the  **root directory**  of the  `celoSmartContract` folder. To do so, navigate to the root directory of your project and type the following command into your terminal:

```touch .env```

We're going to use [Datahub](https://figment.io/datahub/) to connect to the Celo test network. If you don't have an account, sign up on the [Datahub](https://figment.io/datahub/) website and resume this tutorial once you have your Celo API key. 

Next, open the .env file in your text editor and add the following variable:

`REST_URL=https://celo-alfajores--rpc.datahub.figment.io/apikey/<YOUR API KEY>/`

**Note:**  there needs to be a trailing / at the end for this to work!

# Getting a Celo account

Next, we’re going to need a Celo account to deploy from. We will need three things for deployment:

-   A Celo account address
-   A Celo account private key
-   A Celo account  [loaded with testnet funds](https://celo.org/developers/faucet)

First things first, let's get an account and a private key. Create a file named **getAccount.js** in your project folder. In that file, write the following: 

```
const ContractKit =  require('@celo/contractkit');

const Web3 =  require('web3');

require('dotenv').config();

const main =  async  ()  =>  {
  const web3 =  new  Web3(process.env.REST_URL);
  const client = ContractKit.newKitFromWeb3(web3);

  const account = web3.eth.accounts.create();

  console.log('address: ', account.address);
  console.log('privateKey: ', account.privateKey);

};

main().catch((err)  =>  {

  console.error(err);

});
```

Next, run the code in your terminal: 

``node getAccount.js``

Your output should look something like this: 

![output](https://i.imgur.com/uq1LXTf.png)

**Note:** It is important to keep your private key hidden! Whoever has it can access all your funds. 

Now that you have a Celo account, copy the privateKey into your .env file: 

``PRIVATE_KEY=YOUR-PRIVATE-KEY``

Next, take the account address and paste it into the [Celo developer faucet](https://celo.org/developers/faucet). This will give you testnet funds you can use to deploy your smart contract. Fill out the form and wait a couple of seconds, and your account should be loaded up and ready to go.

![funding](https://i.imgur.com/zPtWWHW.png)

# Truffle config

The **truffle-config.js** file is used in order to tell truffle how you want to deploy your contract.

For our purposes, write the following in your truffle-config file: 

```
const ContractKit = require('@celo/contractkit');
const Web3 = require('web3');

require('dotenv').config({path: '.env'});

// Create connection to DataHub Celo Network node
const web3 = new Web3(process.env.REST_URL);
const client = ContractKit.newKitFromWeb3(web3);

// Initialize account from our private key
const account = web3.eth.accounts.privateKeyToAccount(process.env.PRIVATE_KEY);

// We need to add private key to ContractKit in order to sign transactions
client.addAccount(account.privateKey);

module.exports = {
  compilers: {
    solc: {
      version: "0.6.6",    // Fetch exact version from solc-bin (default: truffle's version)
    }
  },
  networks: {
    test: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*"
    },
    alfajores: {
      provider: client.connection.web3.currentProvider, // CeloProvider
      network_id: 44787  // latest Alfajores network id
    }
  }
};
```

This will allow truffle to deploy the alfajores Celo test network. 

# Deployment

We’re almost there! Run the following to check that you did everything correctly:

`truffle compile`

If things are working, you should see the following output:

![compile output](https://i.imgur.com/h5Y6rco.png)

Now that we’ve compiled the smart contract, the last step is to deploy it. Run the following to deploy to the Alfajores testnet:

`truffle migrate --network alfajores`

You should see the following deployment output:

![output](https://i.imgur.com/h7WwqaD.png)

If you see something like the above, you did it correctly! To see your smart contract on the Celo network, open the [Alfajores block explorer](https://alfajores-blockscout.celo-testnet.org/) and paste in the address on the contract address line of your truffle output:  

 ```> contract address:    0xc13891Df18E57137c40876ad206a9B0A30dF8CF5```

You should see a successful contract deployment at that address! 

![block explorer](https://i.imgur.com/nl288ia.png)

# Conclusion

Congrats! You've just deployed a smart contract on the Celo network. 

Now that you've finished the tutorial, you should have a basic understanding of deploying smart contracts on the Celo network. The possibilities are endless for what you can create! It's still early. You can use this tutorial as a jumping off point for deploying the smart contracts of your dreams 🥳

The complete source code for this tutorial can be found on  [Github](https://github.com/alexreyes/Celo-Deploying-With-Truffle).

# Next steps

Now that you've learned how to deploy smart contracts on Celo, you can build new use cases for the cryptoeconomy on Celo. Feel free to learn more solidity, or continue the tutorials on [Figment Learn](https://learn.figment.io).

# Common Errors

If you run into errors at any point, feel free to ask on the Celo channel on the Figment Learn discord server. In any case, here's are some common errors you might face.

---
If you get the following error: 

``Error: Invalid JSON RPC response: {"message":"no Route matched with those values"}
``

![json rpc error](https://i.imgur.com/B8LerrU.png)

Then it's a problem with the **REST_URL** in your **.env** file. 

Make sure the URL has a trailing **/** at the end! It should look like this: 

``REST_URL = https://celo-alfajores--rpc.datahub.figment.io/apikey/YOUR-API-KEY/``

---
If your contract didn't deploy to the test network, your output might look like this: 

![didn't deploy](https://i.imgur.com/p67dZDM.png)

Where the contracts compiled but it didn't give you an address for where it was deployed to.

To fix this, make sure you have your account loaded up with [testnet funds](https://celo.org/developers/faucet). 

If you want to double check that your account received the funds, go to the [Alfajores block explorer](https://alfajores-blockscout.celo-testnet.org/) and paste in your account's address. 

Make sure your account isn't empty like this one!

![empty account](https://i.imgur.com/yPUYCSD.png)
