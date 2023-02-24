---
layout: default
---

# ETHDenver<br>Gasless NFT Marketplace Workshop
{: style="text-align: center; font-size:300%"}

--- 

## Purpose of this workshop
In this workshop we will build an NFT marketplace that will allow us to have users own an NFT without having to own any ETH for transfers.
This will be done by using Meta Transactions, meta transactions can be described as follows (high level):
- We have three users (each with their own), Alice, Bob and Chris. Alice owns an NFT but no ETH, Chris has an empty wallet and Bob has ETH.
- Alice wants to transfer her NFT to Chris, but doesn't have any ETH to cover this
- Alice signs a transaction off-chain for the transfer and sends the transaction and signature to Bob
- Bob creates a transaction which inside of it, the data of Alice's transaction is placed
- Bob sends the transaciton out to the block-chain after signing it
- Contract A verifies the signature data of Alice's transaction
- Assuming signature is ok, Contract A will invoke the NFT contract which will then execute the transfer

## Expanding our operations

In the first part of the workshop we created an NFT collection. This collection unfortunately does not support gasless (/ meta) transactions which we need to make our marketplace.
We will build new parts to our infrastructure with our next steps being:
1. Creating several vaults, and in each create a wallet for ETH_TEST3
2. Re-vamping the contract so that it will also implement [EIP-2771](https://eips.ethereum.org/EIPS/eip-2771) - meta transactions
3. Airdrop one (or several) NFT(s) for each vault account we created in step 1
4. Write and execute code that will perform a transfer of assets between two vault accounts, with our primary - "Treasury" vault account paying for these transfers

### Expanding our workspace

First, let's rename the vault identified by the `id` we recorded in the first activite to `Treasury`. This vault will house all our assets and will be used as the fee payer futher down the line. To do this, review this [API reference](https://developers.fireblocks.com/reference/put_vault-accounts-vaultaccountid).

Now lets expand the workspace, revisit the previous work done when we [created our Fireblocks workspace](./creating-an-nft-on-fireblocks#creating-your-fireblocks-environment). Create several vaults, as many as you'd like. These vaults will act as holders of some tokens from our NFT collection. Be sure to their `id`s, so that we can make use of them later on.

### Creating a meta-transaction Compatible contract

Our original contract does not allow for a transfer of the token between accounts with a different account paying for said transfer. To allow this to be performed, we must change the contract code to support this.

On a high level the following will be done:
1. We will add several files to our contract that will allow us to execute meta transactions
2. Change the original contract to support gasless transactions

Firstly, we add the needed dependencies:
```shell
npm install @opengsn/contracts --save
```

Secondly we add the files:

The following contract will allow us to do just that:
<div>
  <div class="tab">
      <button id="default-open" class="tablinks-meta-tx-sol" onclick="openTab(event, 'meta-tx-sol')" >NativeMetaTransaction.sol</button>
      <button class="tablinks-meta-tx-sol" onclick="openTab(event, 'eip712-base')">EIP712Base.sol</button>
      <button class="tablinks-meta-tx-sol" onclick="openTab(event, 'initable')">Initializable.sol</button>
  </div>

  <div id="meta-tx-sol" class="tabcontent-meta-tx-sol">
      {% highlight dart %}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import {SafeMath} from "@openzeppelin/contracts/utils/math/SafeMath.sol";
import {EIP712Base} from "./EIP712Base.sol";

contract NativeMetaTransaction is EIP712Base {
    using SafeMath for uint256;
    bytes32 private constant META_TRANSACTION_TYPEHASH = keccak256(
        bytes(
            "MetaTransaction(uint256 nonce,address from,bytes functionSignature)"
        )
    );
    event MetaTransactionExecuted(
        address userAddress,
        address payable relayerAddress,
        bytes functionSignature
    );
    mapping(address => uint256) nonces;

    /*
     * Meta transaction structure.
     * No point of including value field here as if user is doing value transfer then he has the funds to pay for gas
     * He should call the desired function directly in that case.
     */
    struct MetaTransaction {
        uint256 nonce;
        address from;
        bytes functionSignature;
    }

    function executeMetaTransaction(
        address userAddress,
        bytes memory functionSignature,
        bytes32 sigR,
        bytes32 sigS,
        uint8 sigV
    ) public payable returns (bytes memory) {
        MetaTransaction memory metaTx = MetaTransaction({
            nonce: nonces[userAddress],
            from: userAddress,
            functionSignature: functionSignature
        });

        require(
            verify(userAddress, metaTx, sigR, sigS, sigV),
            "Signer and signature do not match"
        );

        // increase nonce for user (to avoid re-use)
        nonces[userAddress] = nonces[userAddress].add(1);

        emit MetaTransactionExecuted(
            userAddress,
            payable(msg.sender),
            functionSignature
        );

        // Append userAddress and relayer address at the end to extract it from calling context
        (bool success, bytes memory returnData) = address(this).call(
            abi.encodePacked(functionSignature, userAddress)
        );
        require(success, "Function call not successful");

        return returnData;
    }

    function hashMetaTransaction(MetaTransaction memory metaTx)
        internal
        pure
        returns (bytes32)
    {
        return
            keccak256(
                abi.encode(
                    META_TRANSACTION_TYPEHASH,
                    metaTx.nonce,
                    metaTx.from,
                    keccak256(metaTx.functionSignature)
                )
            );
    }

    function getNonce(address user) public view returns (uint256 nonce) {
        nonce = nonces[user];
    }

    function verify(
        address signer,
        MetaTransaction memory metaTx,
        bytes32 sigR,
        bytes32 sigS,
        uint8 sigV
    ) internal view returns (bool) {
        require(signer != address(0), "NativeMetaTransaction: INVALID_SIGNER");
        return
            signer ==
            ecrecover(
                toTypedMessageHash(hashMetaTransaction(metaTx)),
                sigV,
                sigR,
                sigS
            );
    }
}{% endhighlight %}

  </div>

  <div id="eip712-base" class="tabcontent-meta-tx-sol">
        {% highlight dart %}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import {Initializable} from "./Initializable.sol";

contract EIP712Base is Initializable {
    struct EIP712Domain {
        string name;
        string version;
        address verifyingContract;
        bytes32 salt;
    }

    string constant public ERC712_VERSION = "1";

    bytes32 internal constant EIP712_DOMAIN_TYPEHASH = keccak256(
        bytes(
            "EIP712Domain(string name,string version,address verifyingContract,bytes32 salt)"
        )
    );
    bytes32 internal domainSeperator;

    // supposed to be called once while initializing.
    // one of the contractsa that inherits this contract follows proxy pattern
    // so it is not possible to do this in a constructor
    function _initializeEIP712(
        string memory name
    )
        internal
        initializer
    {
        _setDomainSeperator(name);
    }

    function _setDomainSeperator(string memory name) internal {
        domainSeperator = keccak256(
            abi.encode(
                EIP712_DOMAIN_TYPEHASH,
                keccak256(bytes(name)),
                keccak256(bytes(ERC712_VERSION)),
                address(this),
                bytes32(getChainId())
            )
        );
    }

    function getDomainSeperator() public view returns (bytes32) {
        return domainSeperator;
    }

    function getChainId() public view returns (uint256) {
        uint256 id;
        assembly {
            id := chainid()
        }
        return id;
    }

    /**
     * Accept message hash and returns hash message in EIP712 compatible form
     * So that it can be used to recover signer from signature signed using EIP712 formatted data
     * https://eips.ethereum.org/EIPS/eip-712
     * "\\x19" makes the encoding deterministic
     * "\\x01" is the version byte to make it compatible to EIP-191
     */
    function toTypedMessageHash(bytes32 messageHash)
        internal
        view
        returns (bytes32)
    {
        return
            keccak256(
                abi.encodePacked("\x19\x01", getDomainSeperator(), messageHash)
            );
    }
}{% endhighlight %}
  </div>
  <div id="initable" class="tabcontent-meta-tx-sol">
        {% highlight dart %}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

contract Initializable {
    bool inited = false;

    modifier initializer() {
        require(!inited, "already inited");
        _;
        inited = true;
    }
}{% endhighlight %}
  </div>

  <script>
    function openTab(evt, t_name) {
      var i, tabcontent, tablinks;
      tabcontent = document.getElementsByClassName("tabcontent-meta-tx-sol");
      for (i = 0; i < tabcontent.length; i++) {
        tabcontent[i].style.display = "none";
      }
      tablinks = document.getElementsByClassName("tablinks-meta-tx-sol");
      for (i = 0; i < tablinks.length; i++) {
        tablinks[i].className = tablinks[i].className.replace(" active", "");
      }
      document.getElementById(t_name).style.display = "block";
      evt.currentTarget.className += " active";
    }
    document.getElementById("default-open").click();
  </script>
</div>

The contract content of each file can be copied and saved under `./contracts/` and the relevant name in the tab.

We will briefly explain the purpose of each file:
1. `Initializable.sol` - simply indicates when something that has the `initalizable` modifier, has already been initalized
2. `EIP712Base.sol` - Provides some support for [EIP-712](https://eips.ethereum.org/EIPS/eip-712) message structure. This is used for meta transactions. Considering the above example, this is used to help with verifying that alice indeede signed the transaction.
3. `NativeMetaTransaction.sol` - Provide support for Meta Transactions by exposing the `executeMetaTransaction` function and handling both exectuion and verification of such a transaction.


Finally, we can now create our NFT collection that will support meta transactions, as follows:
```dart
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import "@openzeppelin/contracts/metatx/ERC2771Context.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@opengsn/contracts/src/ERC2771Recipient.sol";
import "./NativeMetaTransaction.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract RobotDoomsday is ERC721,
    ERC721URIStorage,
    Pausable,
    Ownable,
    ERC2771Recipient,
    EIP712Base,
    NativeMetaTransaction {
    using Counters for Counters.Counter;

    Counters.Counter private _tokenIdCounter;

    constructor(address trustedForwarder) ERC721("RobotDoomsday", "RDD") {
        _setTrustedForwarder(trustedForwarder);
        _initializeEIP712("RobotDoomsday");
    }

    /// @inheritdoc IERC2771Recipient
    // Overriden to support both ERC2771Recipient and NativeMetaTransaction
    function isTrustedForwarder(address forwarder)
        public
        view
        virtual
        override
        returns (bool)
    {
        return
            ERC2771Recipient.isTrustedForwarder(forwarder) ||
            forwarder == address(this);
    }

    function _msgSender()
        internal
        view
        override(Context, ERC2771Recipient)
        returns (address sender)
    {
        return ERC2771Recipient._msgSender();
    }

    function _msgData()
        internal
        view
        override(Context, ERC2771Recipient)
        returns (bytes calldata)
    {
        return ERC2771Recipient._msgData();
    }
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

The contract is fairly similar, however it now has several functions that support the execution of Meta Transactions.


### Deploying our contract

Similar to the previous deployment, we will be using the `deploy.js` script to deploy the contract.
In contract to before, this time our constructor is receiving a variable called `trustedForwarder`, which is an address of an entity known as a `TrustedForwarder`. Generally this is a part of the Meta Transaction exeuction flow and we will not be going into exactly what it is or how it works, more information can be found in [EIP-2771](https://eips.ethereum.org/EIPS/eip-2771).
The trusted forwarder we will be using is one from OpenGSN, which can be found [here](https://docs.opengsn.org/networks/ethereum/goerli.html) - 0xB2b5841DBeF766d4b521221732F9B618fCf34A87

This is the updated deploy script:
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
  const robotDoomsday = await RobotDoomsday.deploy("0xB2b5841DBeF766d4b521221732F9B618fCf34A87");

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

Let's execute the deploy script:
```shell
npx hardhat run --network goerli scripts/deploy.js
```

You will get the following message:
`RobotDoomsday deployed to: <contract_address>`

### Airdropping our NFTs

Now we want to create an air-drop at least one NFT to one of the vault accounts we created [before](#expanding-our-workspace);
This can be done in one of two ways:
1. Create a new .js file, and use the `FireblocksWeb3Provider` as well as `ethers` to run the mint function
2. Use the `mint.js` we created previously to mint new NFTs

We will be using the `mint.js` file;
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
  const destAddress = "<TARGET_ADDRESS>"
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

  const tx = await robotDoomsday.safeMint(destAddress, tokenURI);
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

Before you finish editing the file, make sure to replace the `robotDoomsdayAddress` (line 12), `<TARGET_ADDRESS>` (line 15) and `IMAGE_URL` (line 23).
For the `destAddress` be sure to pick an address from one of the vaults you created [previously](#expanding-our-workspace)
The first is the address you recorded when performing the creation of the contract [here](#deploy-your-contract).
The second is the address that will get the NFT airdropped to them.
The third is an image URI to use, [here](./nft-image-table) is a table with images to use.

We will now mint the NFT;

```shell
npx hardhat run --network goerli scripts/mint.js
```

Similar to the creation, this might take a bit of time as a transaction needs to be signed and broadcasted.
At the end of it you will receive the following message:
`A new survivor has been found at: <eth_address>`

You can perform the `getOwnedNFTs` method from the previous activity [here](./creating-an-nft-on-fireblocks#getting-the-token-data). 

---

## Creating a gasless transaction

We will now show how to perform a gasless transaction.
To do this we will setup `ethers` with the `FireblocksWeb3Provider` and execute a `safeTransferFrom` function call. Due to the way we will initialize the FireblocksWeb3Provider we will see a gasless transaction being performed, speficially, our treasury vault account will run the `executeMeatTransaction` function on our contract.

### Setup the provider

First we need to add a single dependency:
```shell
npm install ethers@5.6.9
```

Now we will setup our provider, within a new file we will create the provider:
```javascript
const fs = require('fs');
const path = require('path');
const { FireblocksWeb3Provider, ApiBaseUrl } = require('@fireblocks/fireblocks-web3-provider');
//...
const apiSecret = fs.readFileSync(path.resolve("<path-to-fireblocks_secret.key>"), "utf8");
const apiKey = "<api-key>"

//...
const fbksProvider = new FireblocksWeb3Provider({
    apiKey,
    apiBaseUrl: ApiBaseUrl.Sandbox,
    privateKey: apiSecret,
    gaslessGasTankVaultId: "<TREASURY-VAULT-ACCOUNT-ID>",
    rpcUrl: "https://rpc.ankr.com/eth_goerli",
    vaultAccountIds: ["<NFT-OWNER-VAULT-ACCOUNT-ID>"]
});

```

The `<TREASURY-VAULT-ACCOUNT-ID>` is the id of the vault account whose name we changed before, and that contains all our ETH, `<NFT-OWNER-VAULT-ACCOUNT-ID>` is the id of the vault account to which we air-dropped an NFT.

By the addition of the `gaslessGasTankVaultId` we configured the provider to know that all next transfers are to be done in a gasless transaction manner (i.e. meta transaction).

### Performing a gasless transfer

First we must obtain our contract's ABI, this is very simple to do, simply go to the hardhat project folder, after the deployment you should see an artifacts folder, enter it and then enter contracts and finally enter the folder named `robotdoomsday.sol`, inside you will see a file called `RobotDoomsday.json`.
Open this file and copy the contents of the `abi` field (the array without the key).
Now go to the folder with your code and create a file called `contract-abi.json`, into it paste the data.

Now, the last part is to create a transaction.
Both ethers and web3 accept a provider in the constructor, so you can use whichever one you prefer.

We will show this done using `ethers`:
```javascript
const abi = require('./contract-abi.json');

//...
const provider = new ethers.providers.Web3Provider(fbksProvider);
const signer = provider.getSigner('<NFT-OWNER-ADDRESS>')

const robotDoomsdayContract = new ethers.Contract("<NFT-CONTRACT-ADDRESS", abi, signer)
const res = await robotDoomsdayContract.safeTransferFrom("<NFT-OWNER-ADDRESS>", "<NEW-NFT-OWNER-ADDRESS>", TOKEN-ID);
console.log(await res.wait())
```

You will need to change the relevant components, to their corresponding value. The `TOKEN-ID` field can be obtained from the console, API or etherscan.

Once this is run you will see that on the blockchain the NFT has changed ownership. You can also see this in the console's NFT gallery.

This is the complete file's code:
```javascript
const fs = require('fs');
const path = require('path');
const { inspect } = require('util');
const { FireblocksWeb3Provider, ApiBaseUrl } = require('@fireblocks/fireblocks-web3-provider');
const { Web3 } = require("web3-js");
const { ethers } = require("ethers")
const abi = require('./contract-abi.json');

const apiSecret = fs.readFileSync(path.resolve("</path/to/fireblocks_secret.key>"), "utf8");
const apiKey = "<api-key>"

(async () => {

  const fbksProvider = new FireblocksWeb3Provider({
    apiKey,
    apiBaseUrl: "https://sandbox-api.fireblocks.io",
    privateKey: apiSecret,
    gaslessGasTankVaultId: "<TREASURY-VAULT-ACCOUNT-ID>",
    rpcUrl: "https://rpc.ankr.com/eth_goerli",
    vaultAccountIds: ["<NFT-OWNER-VAULT-ACCOOUNT-ID>"]
  });

  const provider = new ethers.providers.Web3Provider(fbksProvider);
  const signer = provider.getSigner('<NFT-OWNER-ADDRESS>');

  const GaslessToken = new ethers.Contract("<NFT-CONTRACT-ADDRESS>", abi, signer)
  const res = await GaslessToken.safeTransferFrom("<NFT-OWNER-ADDRESS>", "<NEW-OWNER-ADDRESS>", <TOKEN-ID>);
  console.log(inspect(res, false, null, true));

  console.log(await res.wait())
})()
```