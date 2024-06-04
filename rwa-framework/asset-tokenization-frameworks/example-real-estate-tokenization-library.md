# Example: Real Estate Tokenization Library

To illustrate how Evire's Asset Tokenization libraries can be utilized in a smart contract, we'll provide an example library that facilitates the tokenization of real-world assets. This example will focus on a basic implementation for tokenizing real estate assets.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract RealEstateToken is ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    struct Property {
        uint256 id;
        string location;
        uint256 valuation;
        bool forSale;
    }

    mapping(uint256 => Property) public properties;

    event PropertyTokenized(uint256 tokenId, string location, uint256 valuation, address owner);
    event PropertyForSale(uint256 tokenId, uint256 valuation);
    event PropertySold(uint256 tokenId, address newOwner, uint256 salePrice);

    constructor() ERC721("RealEstateToken", "RET") {}

    function tokenizeProperty(string memory _location, uint256 _valuation) public onlyOwner returns (uint256) {
        _tokenIds.increment();
        uint256 newPropertyId = _tokenIds.current();

        _mint(msg.sender, newPropertyId);
        _setTokenURI(newPropertyId, _location);

        properties[newPropertyId] = Property({
            id: newPropertyId,
            location: _location,
            valuation: _valuation,
            forSale: false
        });

        emit PropertyTokenized(newPropertyId, _location, _valuation, msg.sender);
        return newPropertyId;
    }

    function setForSale(uint256 _tokenId, uint256 _valuation) public onlyOwnerOf(_tokenId) {
        Property storage property = properties[_tokenId];
        property.forSale = true;
        property.valuation = _valuation;

        emit PropertyForSale(_tokenId, _valuation);
    }

    function buyProperty(uint256 _tokenId) public payable {
        Property storage property = properties[_tokenId];
        require(property.forSale, "Property not for sale");
        require(msg.value >= property.valuation, "Insufficient funds");

        address previousOwner = ownerOf(_tokenId);
        _transfer(previousOwner, msg.sender, _tokenId);

        property.forSale = false;

        payable(previousOwner).transfer(msg.value);

        emit PropertySold(_tokenId, msg.sender, msg.value);
    }

    modifier onlyOwnerOf(uint256 _tokenId) {
        require(ownerOf(_tokenId) == msg.sender, "Not the owner");
        _;
    }
}
```

#### Explanation

* We use OpenZeppelin's ERC721 standard to represent the real estate as non-fungible tokens (NFTs). The `Ownable` contract ensures that only the contract owner can perform certain actions.
* This struct holds details about each property, including its unique ID, location, valuation, and sale status.
* The `tokenizeProperty` function allows the owner to create a new property token. This function mints a new ERC721 token, sets its metadata (location), and stores the property details.
* The `setForSale` function allows the owner to mark a property as available for sale and set its valuation.
* The `buyProperty` function allows someone to purchase a property that is for sale by sending the appropriate amount of Ether. The ownership of the token is transferred, and the previous owner is paid.

### Example of Usage:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./RWAFW/RealEstateToken.sol";

contract ExtendedRealEstateToken is RealEstateToken {
    struct CoOwner {
        address owner;
        uint256 share; // Percentage of ownership
    }

    mapping(uint256 => CoOwner[]) public coOwners;
    mapping(uint256 => uint256) public rentalIncome;
    mapping(uint256 => mapping(address => uint256)) public withdrawnDividends;

    event DividendWithdrawn(uint256 tokenId, address coOwner, uint256 amount);
    event BidPlaced(uint256 tokenId, address bidder, uint256 bidAmount);
    event BidAccepted(uint256 tokenId, address previousOwner, address newOwner, uint256 bidAmount);

    function addCoOwner(uint256 _tokenId, address _coOwner, uint256 _share) public onlyOwnerOf(_tokenId) {
        coOwners[_tokenId].push(CoOwner({
            owner: _coOwner,
            share: _share
        }));
    }

    function distributeRentalIncome(uint256 _tokenId, uint256 _amount) public onlyOwnerOf(_tokenId) {
        rentalIncome[_tokenId] += _amount;
    }

    function withdrawDividend(uint256 _tokenId) public {
        uint256 totalIncome = rentalIncome[_tokenId];
        require(totalIncome > 0, "No income to withdraw");

        uint256 ownerShare;
        for (uint256 i = 0; i < coOwners[_tokenId].length; i++) {
            if (coOwners[_tokenId][i].owner == msg.sender) {
                ownerShare = coOwners[_tokenId][i].share;
                break;
            }
        }

        uint256 withdrawableAmount = (totalIncome * ownerShare / 100) - withdrawnDividends[_tokenId][msg.sender];
        require(withdrawableAmount > 0, "No dividends to withdraw");

        withdrawnDividends[_tokenId][msg.sender] += withdrawableAmount;
        payable(msg.sender).transfer(withdrawableAmount);

        emit DividendWithdrawn(_tokenId, msg.sender, withdrawableAmount);
    }

    function placeBid(uint256 _tokenId) public payable {
        require(properties[_tokenId].forSale, "Property not for sale");
        require(msg.value > properties[_tokenId].valuation, "Bid is too low");

        emit BidPlaced(_tokenId, msg.sender, msg.value);
    }

    function acceptBid(uint256 _tokenId, address _bidder, uint256 _bidAmount) public onlyOwnerOf(_tokenId) {
        require(properties[_tokenId].forSale, "Property not for sale");
        require(_bidAmount > properties[_tokenId].valuation, "Bid amount is too low");

        address previousOwner = ownerOf(_tokenId);
        _transfer(previousOwner, _bidder, _tokenId);

        properties[_tokenId].forSale = false;

        payable(previousOwner).transfer(_bidAmount);

        emit BidAccepted(_tokenId, previousOwner, _bidder, _bidAmount);
    }

    function getCoOwners(uint256 _tokenId) public view returns (CoOwner[] memory) {
        return coOwners[_tokenId];
    }
}
```

#### Explanation

* The `CoOwner` struct allows multiple addresses to own a share of the property. The `addCoOwner` function assigns shares to different co-owners.
* The contract manages rental income distribution through `distributeRentalIncome` and allows co-owners to withdraw their share of the rental income using `withdrawDividend`.
* The contract supports a bidding mechanism where potential buyers can place bids on properties that are for sale, and owners can accept these bids through `placeBid` and `acceptBid`.
* Events like `DividendWithdrawn`, `BidPlaced`, and `BidAccepted` provide a way to track important actions within the contract.

This example demonstrates how to build a real estate management system using Solidity and the `RealEstateToken.sol` library. It covers tokenization, co-ownership, rental income distribution and a bidding system, showcasing the versatility and power of Solidity for creating advanced blockchain applications.
