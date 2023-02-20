---
layout: default
---

# ETHDenver - Web Shop Workshop
{: style="text-align: center; font-size:300%"}

## QR to this page
{: style="text-align: center; font-size:200%"}

![Image.png](https://res.craft.do/user/full/caac9a89-985a-6c2e-405b-af5f5b2a3ca2/doc/A872B62F-7886-41C9-93D9-60B1D87FD656/5345D40E-6659-4008-889C-547D986079EE_2/Yb82NAokQg3WtswZhIpiPvUJEQx2kuTJzgZ8cYpnkRoz/Image.png)
{: style="text-align: center; margin:0"}
Password: `W3bSh0p`
{: style="text-align: center"}

---

## Table of Contents

1. [Registering to ETHDenver Sandbox](#sign-up-to-sandbox)
2. [Creating your Fireblocks Environment](#creating-your-fireblocks-environment)
   1. [Setup the Fireblocks SDK](#setup-the-fireblocks-sdk)
   2. [Creating a vault account](#create-a-vault-account)
   3. [Creating a wallet](#create-a-wallet)
3. [Fund you wallet](#fund-your-wallet)
   1. [Query your current balance](#verify-your-balance)
   2. [Obtain funds from a faucet](#fund-your-wallet)
4. [Deploy an NFT Collection](#deploy-an-nft-collection)
   1. [Install hardhat and create a project](#install-hardhat-and-create-a-project)
   2. [Install hardhat dependencies](#install-hardhat-dependencies)
   3. [Adjust hardhat config file](#configure-the-fireblocks-plugin)
   4. [Deploy the NFT contract](#deploy-your-contract)
5. [Mint your first token](#mint-your-first-token)
   1. [Creating the mint script](#mint-script)
   2. [Minting the token](#minting-your-token)
   3. [Refreshing NFT data](#getting-the-token-data)

---

## Sign up to Sandbox

Use the following [link](https://info.fireblocks.com/ethdenver-sandbox?hs_preview=ZIKteoCh-101565768816) to register for ETHDenver developer Sandbox.

See the full [Fireblocks API Documentation](https://developers.fireblocks.com/reference/api-overview) for available functions.

Make sure you have your prefered IDE installed, our recomendation is [PyCharm](https://www.jetbrains.com/pycharm/) for python and [Visual Studio Code](https://code.visualstudio.com/) for JS/TS.

For the rest of the workshop, please make sure you have installed (python users also need to install NodeJS):

- [Python](https://www.python.org/downloads/) - Python version >=3.6
- JS/[TS](https://www.typescriptlang.org/download) - NodeJS version >=12

Once installed, install the following (python users also need to install the JS/TS packages):

- For Python specific users: `fireblocks-sdk`
- For JS/TS: `fireblocks-sdk`, `@fireblocks/fireblocks-json-rpc`, `@fireblocks/fireblocks-web3-provider`

For your convinience, commands to run:

```shell
# Python only
pip3 install fireblocks-sdk

# Python and JS/TS
npm install @fireblocks/fireblocks-web3-provider
npm install -g @fireblocks/fireblocks-json-rpc

# JS/TS only
npm install fireblocks-sdk
```

---

## Creating your Fireblocks Environment

### Setup the Fireblocks SDK

Througout the workshop you will need to use the Fireblocks SDK to perform operations (including API calls), the following step shows how to set up the SDK;

<div>
  <div class="tab">
      <button class="tablinks-create-va" onclick="openCreateVATab(event, 'py-create-va')" id="defaultOpen_create_va">Python</button>
      <button class="tablinks-create-va" onclick="openCreateVATab(event, 'js-create-va')">JS</button>
  </div>

  <div id="py-create-va" class="tabcontent-create-va">
      {% highlight python %}from fireblocks_sdk import FireblocksSDK

api_secret = open('</path/to/fireblocks_secret.key>', 'r').read()
api_key = '<your-api-key-here>'
fireblocks = FireblocksSDK(api_secret, api_key)

# If you are working with a Sandbox environment, add the sandbox URL under api_base_u{% endhighlight %}

  </div>

  <div id="js-create-va" class="tabcontent-create-va">
        {% highlight javascript %}const fs = require('fs');
const path = require('path');
const { FireblocksSDK } = require('fireblocks-sdk');

const apiSecret = fs.readFileSync(path.resolve("</path/to/fireblocks_secret.key>"), "utf8");
const apiKey = "<your-api-key-here>"
const fireblocks = new FireblocksSDK(apiSecret, apiKey);{% endhighlight %}

  </div>

  <script>
    function openCreateVATab(evt, t_name) {
      var i, tabcontent, tablinks;
      tabcontent = document.getElementsByClassName("tabcontent-create-va");
      for (i = 0; i < tabcontent.length; i++) {
        tabcontent[i].style.display = "none";
      }
      tablinks = document.getElementsByClassName("tablinks-create-va");
      for (i = 0; i < tablinks.length; i++) {
        tablinks[i].className = tablinks[i].className.replace(" active", "");
      }
      document.getElementById(t_name).style.display = "block";
      evt.currentTarget.className += " active";
    }
    document.getElementById("defaultOpen_create_va").click();
  </script>
</div>

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
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888


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
        vaultAccountIds: "<the id from the account creation",
      },
    },
  },
};
```

Note; python requires additional steps covered later on.

### Deploy your Contract

The following Solidity code is our SpaceBunnies collection's code.

```dart
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract SpaceBunnies is ERC721, ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;

    Counters.Counter private _tokenIdCounter;

    constructor() ERC721("SpaceBunnies", "SPB") {}

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

From within the project's directory, create a new file under the contracts folder, at `./contracts/spacebunnies.sol` and paste the content into it.

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
  const SpaceBunnies = await hre.ethers.getContractFactory("SpaceBunnies");
  const spaceBunnies = await SpaceBunnies.deploy();

  await spaceBunnies.deployed();

  console.log("SpaceBunnies deployed to:", spaceBunnies.address);
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
`SpaceBunnies deployed to: <contract_address>`

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
  const spaceBunniesAddress = "<CONTRACT_ADDRESS>";
  const signer = await hre.ethers.getSigner();
  const signerAdderss = await signer.getAddress();
  const spaceBunnies = await hre.ethers.getContractAt(
    "SpaceBunnies",
    spaceBunniesAddress,
    signer
  );
  const tokenData = {
    name: "SpaceBunny #1",
    image: "<IMAGE_URL>",
  };

  const tokenURI = parser.format(".json", JSON.stringify(tokenData)).content;

  const tx = await spaceBunnies.safeMint(signerAdderss, tokenURI);
  await tx.wait();

  console.log("A new Space Bunny NFT has been minted to:", signerAdderss);
  // console.log("tokenURI:", await spaceBunnies.tokenURI(0))
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Save the file as `mint.js`, with the above content.
Before you finish editing the file, make sure to replace the `spaceBunniesAddress` (line 12) & `IMAGE_URL` (line 18).
The first is the address you recorded when performing the creation of the contract [here](#deploy-your-contract).
The second can be picked from the pre-prepared list:

| Space Bunny                                                                                                              | Image URL                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [\#1](https://serving.photos.photobox.com/86775839cfae69ec4d4530e0bfcfc654521265d2dcc650fc731a815c964fc30b4588a482.jpg)  | [https://serving.photos.photobox.com/86775839cfae69ec4d4530e0bfcfc654521265d2dcc650fc731a815c964fc30b4588a482.jpg](https://serving.photos.photobox.com/86775839cfae69ec4d4530e0bfcfc654521265d2dcc650fc731a815c964fc30b4588a482.jpg) |
| [\#2](https://serving.photos.photobox.com/77936888183b77d01938b843b9159ac9772a7ef6ea80924be04b648a1f2b80e269ecdc0a.jpg)  | [https://serving.photos.photobox.com/77936888183b77d01938b843b9159ac9772a7ef6ea80924be04b648a1f2b80e269ecdc0a.jpg](https://serving.photos.photobox.com/77936888183b77d01938b843b9159ac9772a7ef6ea80924be04b648a1f2b80e269ecdc0a.jpg) |
| [\#3](https://serving.photos.photobox.com/90283830396d3b25b03fd0b97d888a91b5915da69ea0324a13f96884870968d011f80677.jpg)  | [https://serving.photos.photobox.com/90283830396d3b25b03fd0b97d888a91b5915da69ea0324a13f96884870968d011f80677.jpg](https://serving.photos.photobox.com/90283830396d3b25b03fd0b97d888a91b5915da69ea0324a13f96884870968d011f80677.jpg) |
| [\#4](https://serving.photos.photobox.com/56379764a33ad1c911ecc0c2eb82f31f8df7d5b6e1c591dcb903141ef4547ab9e0c1942a.jpg)  | [https://serving.photos.photobox.com/56379764a33ad1c911ecc0c2eb82f31f8df7d5b6e1c591dcb903141ef4547ab9e0c1942a.jpg](https://serving.photos.photobox.com/56379764a33ad1c911ecc0c2eb82f31f8df7d5b6e1c591dcb903141ef4547ab9e0c1942a.jpg) |
| [\#5](https://serving.photos.photobox.com/647134999c5499624ee9507059d105c54ea4fad372e2615e17c49285894df611ec8f40c2.jpg)  | [https://serving.photos.photobox.com/647134999c5499624ee9507059d105c54ea4fad372e2615e17c49285894df611ec8f40c2.jpg](https://serving.photos.photobox.com/647134999c5499624ee9507059d105c54ea4fad372e2615e17c49285894df611ec8f40c2.jpg) |
| [\#6](https://serving.photos.photobox.com/960655291873869a46cb811be2e7ce920538dc0c384b58d1e9b0b579f8a91d0d9be0623b.jpg)  | [https://serving.photos.photobox.com/960655291873869a46cb811be2e7ce920538dc0c384b58d1e9b0b579f8a91d0d9be0623b.jpg](https://serving.photos.photobox.com/960655291873869a46cb811be2e7ce920538dc0c384b58d1e9b0b579f8a91d0d9be0623b.jpg) |
| [\#7](https://serving.photos.photobox.com/353477927fd735a6c9976c53069a9435760e15b3491367a3046d3bf0b3998d5316759459.jpg)  | [https://serving.photos.photobox.com/353477927fd735a6c9976c53069a9435760e15b3491367a3046d3bf0b3998d5316759459.jpg](https://serving.photos.photobox.com/353477927fd735a6c9976c53069a9435760e15b3491367a3046d3bf0b3998d5316759459.jpg) |
| [\#8](https://serving.photos.photobox.com/0959005049f62889a69e9e5a04e420102670725070ec97afe53b86a5da81170a65a375e6.jpg)  | [https://serving.photos.photobox.com/0959005049f62889a69e9e5a04e420102670725070ec97afe53b86a5da81170a65a375e6.jpg](https://serving.photos.photobox.com/0959005049f62889a69e9e5a04e420102670725070ec97afe53b86a5da81170a65a375e6.jpg) |
| [\#9](https://serving.photos.photobox.com/3667709080f77f0b9563633fd603b946ace2351a507301729c416f15ebdb8effcff38400.jpg)  | [https://serving.photos.photobox.com/3667709080f77f0b9563633fd603b946ace2351a507301729c416f15ebdb8effcff38400.jpg](https://serving.photos.photobox.com/3667709080f77f0b9563633fd603b946ace2351a507301729c416f15ebdb8effcff38400.jpg) |
| [\#10](https://serving.photos.photobox.com/24537655cb7ecff4cff23b053227c019b5a5d3690539792a3509113cb476859c8b26bf48.jpg) | [https://serving.photos.photobox.com/24537655cb7ecff4cff23b053227c019b5a5d3690539792a3509113cb476859c8b26bf48.jpg](https://serving.photos.photobox.com/24537655cb7ecff4cff23b053227c019b5a5d3690539792a3509113cb476859c8b26bf48.jpg) |

### Minting your Token

Now that we finished the script, minting is a very simple operation, all that we need to do is to run the following command from the project's directory:

```shell
npx hardhat run --network goerli scripts/mint.js
```

Similar to the creation, this might take a bit of time as a transaction needs to be signed and broadcasted.
At the end of it you will receive the following message:
`A new Space Bunny NFT has been minted to: <eth_address>`

### Getting the Token Data

Now that we minted the token, we let's query the Fireblocks API to see information about our new NFT.
The below code uses the SDK we setup earlier ([here](#setup-the-fireblocks-sdk)):

<div>
<div class="tab">
      <button class="tablinks-get-nfts" onclick="openGetNFTTab(event, 'py-get-nft')" id="defaultOpen_get_nfts">Python</button>
      <button class="tablinks-get-nfts" onclick="openGetNFTTab(event, 'js-get-nft')">JS</button>
  </div>

  <div id="py-get-nft" class="tabcontent-get-nfts">
      {% highlight python %}print(fireblocks.get_owned_nfts("ETH_TEST3", ["<account_id>"])){% endhighlight %}
  </div>

  <div id="js-get-nft" class="tabcontent-get-nfts">
        {% highlight javascript %}(async () => {
  let ownedNfts = await fireblocks.getOwnedNFTs({
    blockchainDescriptor: "ETH_TEST3",
    vaultAccountIds: ["<account_id>"],
  });
  console.log(ownedNfts);
})().catch((e) => {
  console.error(`Failed: ${e}`);
});{% endhighlight %}
  </div>
  <script>
    function openGetNFTTab(evt, t_name) {
      var i, tabcontent, tablinks;
      tabcontent = document.getElementsByClassName("tabcontent-get-nfts");
      for (i = 0; i < tabcontent.length; i++) {
        tabcontent[i].style.display = "none";
      }
      tablinks = document.getElementsByClassName("tablinks-get-nfts");
      for (i = 0; i < tablinks.length; i++) {
        tablinks[i].className = tablinks[i].className.replace(" active", "");
      }
      document.getElementById(t_name).style.display = "block";
      evt.currentTarget.className += " active";
    }
    document.getElementById("defaultOpen_get_nfts").click();
  </script>
</div>

