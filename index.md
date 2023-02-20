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

asdka

```python
def abs():
  pass
```

---

## Creating your Fireblocks Environment

### Create Your Account

<br>
 
<div>
  <div class="tab">
      <button class="tablinks" onclick="openTab(event, 'py')" id="defaultOpen_create_va">Python</button>
      <button class="tablinks" onclick="openTab(event, 'js')">JS</button>
  </div>

  <div id="py" class="tabcontent">
      {% highlight python %}from fireblocks_sdk import FireblocksSDK

api_secret = open('</path/to/fireblocks_secret.key>', 'r').read()
api_key = '<your-api-key-here>'
fireblocks = FireblocksSDK(api_secret, api_key)
# If you are working with a Sandbox environment, add the sandbox URL under api_base_u{% endhighlight %}
  </div>

  <div id="js" class="tabcontent">
        {% highlight javascript %}const fs = require('fs');
const path = require('path');
const { FireblocksSDK } = require('fireblocks-sdk');

const apiSecret = fs.readFileSync(path.resolve("</path/to/fireblocks_secret.key>"), "utf8");
const apiKey = "<your-api-key-here>" 
const fireblocks = new FireblocksSDK(apiSecret, apiKey);{% endhighlight %}
  </div>

  <script>
    function openTab(evt, t_name) {
      var i, tabcontent, tablinks;
      tabcontent = document.getElementsByClassName("tabcontent");
      for (i = 0; i < tabcontent.length; i++) {
        tabcontent[i].style.display = "none";
      }
      tablinks = document.getElementsByClassName("tablinks");
      for (i = 0; i < tablinks.length; i++) {
        tablinks[i].className = tablinks[i].className.replace(" active", "");
      }
      document.getElementById(t_name).style.display = "block";
      evt.currentTarget.className += " active";
    }
    document.getElementById("defaultOpen_create_va").click();
  </script>
</div>


## 1.2 Create your wallet

- Start by [creating your first vault account](https://developers.fireblocks.com/reference/post_vault-accounts).
  This will create an account that can hold multiple wallets, such as BTC, Ethereum, Polygon, and so many more.

Do note that this endpoint returns an `id`, so make sure to capture it! We will use it in the next step and also later on when we will be dealing with our contract.

- Using the `id` from the last step, it is time to [create your wallet](https://developers.fireblocks.com/reference/post_vault-accounts-vaultaccountid-assetid).
  Do note this endpoint also asks for an `assetId` . For our workshop, we will use `"ETH_TEST3"`, which stands for the Goerli network.

This endpoint will return an `address`. Save this one as well!

---

# 2 Fund Your Wallet

## 2.1 Verify your balance

- Check your balance using the [balance endpoint](https://developers.fireblocks.com/reference/get_vault-accounts-vaultaccountid-assetid). Use it with the id from the previous step, and the assetId - `"ETH_TEST3"`.

## 2.2 Faucet

- If your balance hasn’t been updated automatically, reach out to us with your `address` and we will provide some ETH over to your Goerli wallet.

---

# 3 Deploy an NFT Collection

We will now deploy an NFT collection using a smart contract code that mostly relies on OpenZeppelin standards and the Hardhat tool.

This part will guide you step by step, from configuring the Hardhat application, to the successful deployment.

## 3.1 Install Hardhat

Create a new directory for your project:

```shell
mkdir web3-workshop
cd web3-workshop
```

Install hardhat:

```shell
npm install --save-dev hardhat
```

Last but not least, initialize your project:

```shell
npx hardhat
```

You should have the following output:

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

Make sure your choose `Create a JavaScript project`

Delete the default solidity file that comes within the project. Assuming you are in the directory:

```shell
rm -f ./contracts/Lock.sol
```

## 3.2 Install Hardhat Dependencies

We will install both of the required dependencies in order to compile our contract and deploy it through Fireblocks.

- OpenZeppelin

```shell
npm install @openzeppelin/contracts
```

- Fireblocks Hardhat Plugin

```shell
npm install @fireblocks/hardhat-fireblocks
```

- dataURI (which will be used for our minting script):

```shell
npm install datauri
```

## 3.3 Configure the Fireblocks Plugin

After you have have performed all of the above steps, finalize your preparation by editing the `hardhat.config.js` file:

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

## 3.4 Deploy your Contract

We will now deploy a very cool NFT collection, Space Bunnies.

Copy the content of the following Solidity file:

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

Assuming you’re in the projects directory, create a new file under the contracts folder, at `./contracts/spacebunnies.sol` and paste the content into it.

Before you continue, make sure you compile it:

```shell
npx hardhat compile
```

Next, we will edit the `deploy.js` script, under `./scripts`, copying the following content:

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

Now, just execute the deployment through hardhat! Do note this will take a short while.

```shell
npx hardhat run --network goerli scripts/deploy.js
```

After finalizing the contract deployment, you should receive a message with the contract address:

`SpaceBunnies deployed to: <contract_address>`

You can review the contract under [Etherscan](https://goerli.etherscan.io/).

---

# 4 Mint your First Token

## 4.1 Mint Script

Copy the content of the following script, in order to mint your own Space Bunny:

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

Create a new script file for the minting, ./scripts/mint.js , while pasting the content above. Before you finish editing the file, make sure to replace the `spaceBunniesAddress` (line 12) & `IMAGE_URL` (line 18). You can pick a really cool image from the collection we made for you, down here:

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

## 4.2 Minting your Token

We have our script ready. Let’s mint!

Run the minting script using hardhat:

```shell
npx hardhat run --network goerli scripts/mint.js
```

You should retrieve the following output, once the transaction is done:

`A new Space Bunny NFT has been minted to: <eth_address>`

## 4.3 Getting the Token Data

Last but not least, we can get all of the token data using Fireblocks NFT engine!

If you are using the Python SDK, use the following function in your Python file (from step 1.1):

```python
print(fireblocks.get_owned_nfts("ETH_TEST3", ["<account_id>"]))
```

If you are using the JavaScript SDK, use the following function in your JavaScript file (from step 1.1):

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

You can see many of the token traits, such as:

- `tokenId`
- `standard`
- `metadataURI`
- And much more …
