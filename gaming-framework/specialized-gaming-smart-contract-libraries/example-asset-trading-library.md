# Example: Asset Trading Library

The purpose of this library is to facilitate the creation, management and trading of digital assets in blockchain gaming. This library ensures secure, decentralized asset ownership and trading, enabling players to fully control their assets and engage in economic activities within the game ecosystem.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract EvireAssetTradingLibrary is ERC721, ERC721Enumerable, ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIdCounter;

    struct Asset {
        uint256 id;
        string name;
        string metadataURI;
        uint256 price;
        bool forSale;
    }

    mapping(uint256 => Asset) private assets;

    event AssetCreated(uint256 indexed assetId, string name, string metadataURI, uint256 price);
    event AssetForSale(uint256 indexed assetId, uint256 price);
    event AssetSold(uint256 indexed assetId, address indexed buyer, uint256 price);
    event AssetRemovedFromSale(uint256 indexed assetId);

    constructor() ERC721("EvireAsset", "EVA") {}

    function createAsset(string memory name, string memory metadataURI, uint256 price) public onlyOwner {
        uint256 assetId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        
        assets[assetId] = Asset(assetId, name, metadataURI, price, false);
        
        _mint(msg.sender, assetId);
        _setTokenURI(assetId, metadataURI);

        emit AssetCreated(assetId, name, metadataURI, price);
    }

    function listAssetForSale(uint256 assetId, uint256 price) public onlyOwner {
        require(_exists(assetId), "Asset does not exist");
        require(ownerOf(assetId) == msg.sender, "You do not own this asset");

        assets[assetId].price = price;
        assets[assetId].forSale = true;

        emit AssetForSale(assetId, price);
    }

    function buyAsset(uint256 assetId) public payable {
        require(_exists(assetId), "Asset does not exist");
        require(assets[assetId].forSale, "Asset is not for sale");
        require(msg.value == assets[assetId].price, "Incorrect price");

        address owner = ownerOf(assetId);
        _transfer(owner, msg.sender, assetId);

        payable(owner).transfer(msg.value);
        
        assets[assetId].forSale = false;

        emit AssetSold(assetId, msg.sender, msg.value);
    }

    function removeAssetFromSale(uint256 assetId) public onlyOwner {
        require(_exists(assetId), "Asset does not exist");
        require(assets[assetId].forSale, "Asset is not for sale");

        assets[assetId].forSale = false;

        emit AssetRemovedFromSale(assetId);
    }

    function _beforeTokenTransfer(address from, address to, uint256 tokenId) internal override(ERC721, ERC721Enumerable) {
        super._beforeTokenTransfer(from, to, tokenId);
    }

    function _burn(uint256 tokenId) internal override(ERC721, ERC721URIStorage) {
        super._burn(tokenId);
    }

    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        return super.tokenURI(tokenId);
    }

    function supportsInterface(bytes4 interfaceId) public view override(ERC721, ERC721Enumerable) returns (bool) {
        return super.supportsInterface(interfaceId);
    }
}

```

### **Example Usage**

This dApp will manage a marketplace for unique digital assets where users can create, list for sale, buy and remove assets from sale.

#### Smart Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./GaminFW/EvireAssetTradingLibrary.sol";

contract EvireMarketplace {
    EvireAssetTradingLibrary private assetLibrary;

    event NewAsset(uint256 indexed assetId, string name, string metadataURI, uint256 price);
    event AssetListedForSale(uint256 indexed assetId, uint256 price);
    event AssetPurchased(uint256 indexed assetId, address indexed buyer, uint256 price);
    event AssetDelisted(uint256 indexed assetId);

    constructor(address _assetLibraryAddress) {
        assetLibrary = EvireAssetTradingLibrary(_assetLibraryAddress);
    }

    function createAsset(string memory name, string memory metadataURI, uint256 price) public {
        assetLibrary.createAsset(name, metadataURI, price);
        uint256 assetId = assetLibrary.totalSupply() - 1; // Assuming the new asset ID is the last one created
        emit NewAsset(assetId, name, metadataURI, price);
    }

    function listAssetForSale(uint256 assetId, uint256 price) public {
        assetLibrary.listAssetForSale(assetId, price);
        emit AssetListedForSale(assetId, price);
    }

    function buyAsset(uint256 assetId) public payable {
        uint256 price = assetLibrary.assets(assetId).price; // Assuming a getter function exists in EvireAssetTradingLibrary
        require(msg.value == price, "Incorrect value sent");
        assetLibrary.buyAsset{value: msg.value}(assetId);
        emit AssetPurchased(assetId, msg.sender, price);
    }

    function removeAssetFromSale(uint256 assetId) public {
        assetLibrary.removeAssetFromSale(assetId);
        emit AssetDelisted(assetId);
    }

    function getAssetDetails(uint256 assetId) public view returns (EvireAssetTradingLibrary.Asset memory) {
        return assetLibrary.assets(assetId); // Assuming a getter function exists in EvireAssetTradingLibrary
    }

    function totalAssets() public view returns (uint256) {
        return assetLibrary.totalSupply();
    }
}
```

#### Web3.js Integration

Below is a JavaScript example using Web3.js to interact with the deployed `EvireMarketplace` smart contract. This will include functions to create an asset, list it for sale, buy it and remove it from sale.

```javascript
const Web3 = require('web3');
const web3 = new Web3('http://localhost:8545'); // Change to your Ethereum node URL

const marketplaceAbi = [...] // ABI of the EvireMarketplace contract
const marketplaceAddress = '0x...'; // Deployed address of the EvireMarketplace contract

const marketplace = new web3.eth.Contract(marketplaceAbi, marketplaceAddress);

// Example account addresses
const owner = '0x...'; // The owner address
const buyer = '0x...'; // The buyer address

// Create a new asset
async function createAsset(name, metadataURI, price) {
    const priceInWei = web3.utils.toWei(price, 'ether');
    await marketplace.methods.createAsset(name, metadataURI, priceInWei).send({ from: owner });
    console.log(`Asset created: ${name}`);
}

// List an asset for sale
async function listAssetForSale(assetId, price) {
    const priceInWei = web3.utils.toWei(price, 'ether');
    await marketplace.methods.listAssetForSale(assetId, priceInWei).send({ from: owner });
    console.log(`Asset ${assetId} listed for sale at ${price} ETH`);
}

// Buy an asset
async function buyAsset(assetId) {
    const assetDetails = await marketplace.methods.getAssetDetails(assetId).call();
    const price = assetDetails.price;
    await marketplace.methods.buyAsset(assetId).send({ from: buyer, value: price });
    console.log(`Asset ${assetId} bought for ${web3.utils.fromWei(price, 'ether')} ETH`);
}

// Remove an asset from sale
async function removeAssetFromSale(assetId) {
    await marketplace.methods.removeAssetFromSale(assetId).send({ from: owner });
    console.log(`Asset ${assetId} removed from sale`);
}

// Get asset details
async function getAssetDetails(assetId) {
    const asset = await marketplace.methods.getAssetDetails(assetId).call();
    console.log(`Asset ${assetId}:`, asset);
}

// Example usage
(async () => {
    await createAsset('Asset Name', 'https://example.com/metadata.json', '0.1');
    await listAssetForSale(0, '0.1');
    await buyAsset(0);
    await removeAssetFromSale(0);
    await getAssetDetails(0);
})();
```

#### Explanation

1. **Smart Contract**:
   * The `EvireMarketplace` contract initializes with the address of the deployed `EvireAssetTradingLibrary`.
   * It exposes functions to create assets, list them for sale, buy them, and remove them from sale, while emitting corresponding events.
   * It provides a way to get asset details and the total number of assets.
2. **Web3.js Integration**:
   * This script uses Web3.js to interact with the smart contract.
   * It includes functions to create assets, list them for sale, buy them, and remove them from sale.
   * It demonstrates the usage of these functions in an example asynchronous function.

