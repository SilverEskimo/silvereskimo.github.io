---
layout: default
---

# ETHDenver<br>NFT Marketplace Workshop
{: style="text-align: center; font-size:300%"}

--- 

## Sign up to Sandbox

Use the following [link](https://info.fireblocks.com/ethdenver-sandbox) to register for ETHDenver developer Sandbox.
Once you register the system will automatically create an user for you to be able to access the web console and the API.<br/>For registration you'll need an authenticator app as it will be required as part of the sign-up process.

See the full [Fireblocks API Documentation](https://developers.fireblocks.com/reference/api-overview) for available functions.

Make sure you have your prefered IDE installed, our recomendation is [Visual Studio Code](https://code.visualstudio.com/) for JS/TS.

For the rest of the workshop, please make sure you have installed [NodeJS version >=12](https://nodejs.org/en/download/)

Once installed, install the following:
`fireblocks-sdk`, `@fireblocks/fireblocks-json-rpc`, `@fireblocks/fireblocks-web3-provider`

For your convinience, commands to run:

```shell
npm install @fireblocks/fireblocks-web3-provider fireblocks-sdk
npm install -g @fireblocks/fireblocks-json-rpc
```

---

## Creating your Fireblocks Environment

### Create your API User

After registering, you will be brought to the onboarding page, this will show you how to create a new API user.
In the event that you are not brought to this page, you can follow these steps in order to create a new API user.
To create your API user perform the following steps:
1. Click the top-right gear icon next to your name
2. Go to the `Users` tab
3. On the upper middle right side click on `Add User`
4. Switch from `Add User` to `API User`
5. Fill in the name and for the role select `Editor`
6. Select `Automatic CSR`
7. Click on `Add user & Download Private Key`

Now you will have the private key, which will be used for signing the requests.
Now we need to get the API Key, from the same view you're currently on, click on the key icon next to the newly created user.

The API Key wil now be in your clipboard, paste it somewhere for later use.

In addition to the API user, we need to enable One-time address transactions, this is required for contract deployment and interaction. To do this, from your current view (within the settings), go to the `General` tab and scroll to the bottom. You wil see a section titled `One-time Address Transactions`, click `Enable` there.

### Setup the Fireblocks SDK

Througout the workshop you will need to use the Fireblocks SDK to perform operations (including API calls), the following step shows how to set up the SDK;

```javascript
const fs = require('fs');
const path = require('path');
const { FireblocksSDK } = require('fireblocks-sdk');

const apiSecret = fs.readFileSync(path.resolve("</path/to/fireblocks_secret.key>"), "utf8");
const apiKey = "<your-api-key-here>"
const fireblocks = new FireblocksSDK(apiSecret, apiKey, "https://sandbox-api.fireblocks.io");
```

### Create a vault account

Within your workspace, you can have containers for your assets, allowing for logical separation between different sets of assets. Such separation can be used for client needs or asset funneling (for example OTC, or incoming crypto from NFT sales).

These containers are called Vault accounts, each workspace can have an unlimited number of such vault accounts, with each vault containing addresses for various assets (i.e. wallets).
We will now create a vault account via the API to help familiarize ourselves with the API functionality.

To perform this, follow the API reference that can be found [here](https://developers.fireblocks.com/reference/post_vault-accounts).<br>
**This endpoint returns an `id`, make sure to record it as it will be needed later.**

### Create a wallet

Once you have created your vault account, you can create wallets.
Wallets are the basic units within a vault account, each wallet represents a single (or several) address on a given blockchain. Wallets can be made for ETH, BTC, etc.

To do this, follow the API reference that can be found [here](https://developers.fireblocks.com/reference/post_vault-accounts-vaultaccountid-assetid).
The API will require that you provide the vault account `id` which you have recorded in the previous step.
You will also need to provide an asset Id, in the context of this workshop it will be `ETH_TEST3` which is the Goerli tstnet asset id in fireblocks.

**This endpoint will return an `address`, make sure to record it as it will be needed later.**

---

## Fund Your Wallet

### Verify your balance

Throughout your work with the API you will need to understand the state of various vault accounts in your use.
This is done by querying the current asset balance, or in other words the balance of a specific asset's wallet.
To do this, following the relevant API reference, found [here](https://developers.fireblocks.com/reference/get_vault-accounts-vaultaccountid-assetid). This endpoint will require both the `id` and the aforementioned `ETH_TEST3` assetId.

### Obtain funds

Before we proceed we need to fund our wallet, to do this we will need to get funds from a facuet.
In case you are unable to, please reach out to us with thee `address` from the previous step.

---

## Deploy an NFT Collection

We will now deploy an NFT collection. Our collection's smart contract code relies on OpenZeppelin standards and the Hardhat tool.

This part will guide you step by step, from configuring the Hardhat application, to the successful deployment.

### Install Hardhat and create a project

First we will install hardhat; this is a fairly straightforward process.
The following commands will install hardhat and create a new project;

```shell
# Create a new directory for our project
mkdir web3-workshop
cd web3-workshop

# Install hardhat
npm install --save-dev hardhat

# Run hardhat
npx hardhat
```

After the last command you will see the below output

```shell
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888      88b 888P   d88  888 888  88b      88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888  Y888888 888      Y88888 888  888  Y888888   Y888


👷 Welcome to Hardhat v2.10.1 👷‍

? What do you want to do? …
❯ Create a JavaScript project
  Create a TypeScript project
  Create an empty hardhat.config.js
```

Make sure your choose `Create a JavaScript project`.
Now delete the default solidity file that comes within the project. Assuming you are in the directory:

```shell
rm -f ./contracts/Lock.sol
```

### Install Hardhat Dependencies

We will install both of the required dependencies in order to compile our contract and deploy it through Fireblocks.

```shell
# OpenZepplin contracts
npm install @openzeppelin/contracts

# Hardhat Fireblocks plugin
npm install @fireblocks/hardhat-fireblocks

# Data URI will be used in our minting script.
npm install datauri
```

### Configure the Fireblocks Plugin

The last thing we need to do now is to adjust the hardhat configuration file. The following is the prepared hardhat configuration file:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("@fireblocks/hardhat-fireblocks");
const { ApiBaseUrl } = require("@fireblocks/fireblocks-web3-provider");

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.9",
  networks: {
    goerli: {
      url: "https://rpc.ankr.com/eth_goerli",
      fireblocks: {
        apiBaseUrl: ApiBaseUrl.Sandbox,
        privateKey: "<private key location>",
        apiKey: "<your api key>",
        vaultAccountIds: "<the id from the account creation>",
      },
    },
  },
};
```

Note; python requires additional steps covered later on.

### Deploy your Contract

The following Solidity code is our RobotDoomsday collection's code.

```dart
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract RobotDoomsday is ERC721, ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;

    Counters.Counter private _tokenIdCounter;

    constructor() ERC721("RobotDoomsday", "RDD") {}

    function safeMint(address to, string memory uri) public onlyOwner {
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }

    // The following functions are overrides required by Solidity.

    function _burn(uint256 tokenId) internal override(ERC721, ERC721URIStorage) {
        super._burn(tokenId);
    }

    function tokenURI(uint256 tokenId)
        public
        view
        override(ERC721, ERC721URIStorage)
        returns (string memory)
    {
        return super.tokenURI(tokenId);
    }
}
```

From within the project's directory, create a new file under the contracts folder, at `./contracts/robotdoomsday.sol` and paste the content into it.

Before you continue, make sure you compile it:

```shell
npx hardhat compile
```

Next, we will edit the `deploy.js` script under the `./scripts` folder.
This file will act as our deployment script and will actually perform the deployment onto the Ethereum blockchain on the Goerli network.
Copy the following content into the file:

```javascript
// We require the Hardhat Runtime Environment explicitly here. This is optional
// but useful for running the script in a standalone fashion through `node <script>`.
//
// You can also run a script with `npx hardhat run <script>`. If you do that, Hardhat
// will compile your contracts, add the Hardhat Runtime Environment's members to the
// global scope, and execute the script.
const hre = require("hardhat");

async function main() {
  const RobotDoomsday = await hre.ethers.getContractFactory("RobotDoomsday");
  const robotDoomsday = await RobotDoomsday.deploy();

  await robotDoomsday.deployed();

  console.log("RobotDoomsday deployed to:", robotDoomsday.address);
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Now, just execute the deployment through hardhat, this process will require signatures of transactions by you, thus will require some time until it is done.

```shell
npx hardhat run --network goerli scripts/deploy.js
```

After finalizing the contract deployment, you should receive a message with the contract address:
`RobotDoomsday deployed to: <contract_address>`

Make sure to record this adderss as it will be used later on.
You can review the contract under [Etherscan](https://goerli.etherscan.io/).

---

## Mint your First Token

### Mint Script

Once our contract is out in the "wild", we can perform a mint of our token.
To do this, we will make use of hardhat once more - since it is already configured to work with the Fireblocks plugin.
Copy the content of the following script into a new file under `./scripts` folder in order to mint your own Space Bunny:

```javascript
// We require the Hardhat Runtime Environment explicitly here. This is optional
// but useful for running the script in a standalone fashion through `node <script>`.
//
// You can also run a script with `npx hardhat run <script>`. If you do that, Hardhat
// will compile your contracts, add the Hardhat Runtime Environment's members to the
// global scope, and execute the script.
const hre = require("hardhat");
const DatauriParser = require("datauri/parser");
const parser = new DatauriParser();

async function main() {
  const robotDoomsdayAddress = "<CONTRACT_ADDRESS>";
  const signer = await hre.ethers.getSigner();
  const signerAdderss = await signer.getAddress();
  const robotDoomsday = await hre.ethers.getContractAt(
    "RobotDoomsday",
    robotDoomsdayAddress,
    signer
  );
  const tokenData = {
    name: "Robot Survivor",
    image: "<IMAGE_URL>",
  };

  const tokenURI = parser.format(".json", JSON.stringify(tokenData)).content;

  const tx = await robotDoomsday.safeMint(signerAddress, tokenURI);
  await tx.wait();

  console.log("A new survivor has been found at:", destAddress);
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Save the file as `mint.js`, with the above content.
Before you finish editing the file, make sure to replace the `robotDoomsdayAddress` (line 12) & `IMAGE_URL` (line 18).
The first is the address you recorded when performing the creation of the contract [here](#deploy-your-contract).
The second can be picked from the pre-prepared list [here](./nft-image-table):

### Minting your Token

Now that we finished the script, minting is a very simple operation, all that we need to do is to run the following command from the project's directory:

```shell
npx hardhat run --network goerli scripts/mint.js
```

Similar to the creation, this might take a bit of time as a transaction needs to be signed and broadcasted.
At the end of it you will receive the following message:
`A new survivor has been found at: <eth_address>`

### Getting the Token Data

Now that we minted the token, we let's query the Fireblocks API to see information about our new NFT.
The below code uses the SDK we setup earlier ([here](#setup-the-fireblocks-sdk)):
```javascript
(async () => {
  let ownedNfts = await fireblocks.getOwnedNFTs({
    blockchainDescriptor: "ETH_TEST3",
    vaultAccountIds: ["<account_id>"],
  });
  console.log(ownedNfts);
})().catch((e) => {
  console.error(`Failed: ${e}`);
});
```
