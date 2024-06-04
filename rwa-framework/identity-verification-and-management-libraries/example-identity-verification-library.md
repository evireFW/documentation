# Example: Identity Verification Library

This example library, `EvireIdentityVerificationLibrary`, is designed to facilitate identity verification and management for dApps operating in the real-world asset (RWA) domain on the Evire blockchain.&#x20;

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @dev This library provides functions to facilitate identity verification and management for dApps
 *      operating in the real-world asset domain (RWA) using the Evire blockchain.
 */
library EvireIdentityVerificationLibrary {
    
    // Event emitted when a new KYC provider is added
    event KYCProviderAdded(address indexed provider);

    // Event emitted when a user completes the KYC verification process
    event UserVerified(address indexed user, uint256 timestamp);

    struct KYCProvider {
        bool isApproved;
        string name;
        string jurisdiction;
    }

    struct UserKYC {
        bool isVerified;
        uint256 timestamp;
        address verifiedBy;
    }

    mapping(address => KYCProvider) private kycProviders;
    mapping(address => UserKYC) private userKYCRecords;

    /**
     * @dev Adds a new KYC provider to the approved list.
     * @param provider Address of the KYC provider
     * @param name Name of the KYC provider
     * @param jurisdiction Jurisdiction of the KYC provider
     */
    function addKYCProvider(address provider, string memory name, string memory jurisdiction) external {
        require(provider != address(0), "Invalid provider address");
        require(!kycProviders[provider].isApproved, "Provider already approved");

        kycProviders[provider] = KYCProvider({
            isApproved: true,
            name: name,
            jurisdiction: jurisdiction
        });

        emit KYCProviderAdded(provider);
    }

    /**
     * @dev Verifies a user's identity through an approved KYC provider.
     * @param user Address of the user to verify
     */
    function verifyUser(address user) external {
        require(user != address(0), "Invalid user address");
        require(kycProviders[msg.sender].isApproved, "Caller is not an approved KYC provider");

        userKYCRecords[user] = UserKYC({
            isVerified: true,
            timestamp: block.timestamp,
            verifiedBy: msg.sender
        });

        emit UserVerified(user, block.timestamp);
    }

    /**
     * @dev Checks if a user is verified.
     * @param user Address of the user to check
     * @return isVerified Boolean indicating if the user is verified
     * @return timestamp Timestamp when the user was verified
     * @return verifiedBy Address of the KYC provider who verified the user
     */
    function isUserVerified(address user) external view returns (bool isVerified, uint256 timestamp, address verifiedBy) {
        UserKYC storage userRecord = userKYCRecords[user];
        return (userRecord.isVerified, userRecord.timestamp, userRecord.verifiedBy);
    }

    /**
     * @dev Retrieves the details of an approved KYC provider.
     * @param provider Address of the KYC provider
     * @return name Name of the KYC provider
     * @return jurisdiction Jurisdiction of the KYC provider
     */
    function getKYCProviderDetails(address provider) external view returns (string memory name, string memory jurisdiction) {
        require(kycProviders[provider].isApproved, "Provider not approved");

        KYCProvider storage providerDetails = kycProviders[provider];
        return (providerDetails.name, providerDetails.jurisdiction);
    }
    
    /**
     * @dev Utility function to check if a KYC provider is approved.
     * @param provider Address of the KYC provider
     * @return isApproved Boolean indicating if the provider is approved
     */
    function isApprovedProvider(address provider) external view returns (bool isApproved) {
        return kycProviders[provider].isApproved;
    }
}

```

This library includes several essential features:

* Allows authorized entities to add approved KYC providers with specific details like name and jurisdiction.
* Enables these KYC providers to verify users' identities and record the verification details.
* Provides a method to check if a user is verified, including the verification timestamp and the provider who performed the verification.
* Allows retrieval of details about approved KYC providers.
* Includes a utility function to check if a KYC provider is approved.

### Example of Usage

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./RWAFW/EvireIdentityVerificationLibrary.sol";

/**
 * @title RealEstateTokenization
 * @dev This smart contract facilitates the tokenization of real estate properties on the Evire blockchain.
 *      It integrates identity verification using the EvireIdentityVerificationLibrary to ensure compliant transactions.
 */
contract RealEstateTokenization {
    using EvireIdentityVerificationLibrary for address;

    // Struct to represent a real estate property
    struct Property {
        uint256 id;
        string name;
        string location;
        uint256 value;
        address owner;
    }

    // Mapping of property IDs to Property structs
    mapping(uint256 => Property) public properties;
    // Counter for generating unique property IDs
    uint256 private propertyCounter;

    // Event emitted when a new property is tokenized
    event PropertyTokenized(uint256 indexed id, string name, string location, uint256 value, address indexed owner);
    // Event emitted when a property ownership is transferred
    event PropertyTransferred(uint256 indexed id, address indexed from, address indexed to);

    // Modifier to check if the user is verified
    modifier onlyVerifiedUser(address user) {
        (bool isVerified,,) = user.isUserVerified();
        require(isVerified, "User is not verified");
        _;
    }

    /**
     * @dev Tokenizes a new real estate property.
     * @param name Name of the property
     * @param location Location of the property
     * @param value Value of the property
     */
    function tokenizeProperty(string memory name, string memory location, uint256 value) external onlyVerifiedUser(msg.sender) {
        propertyCounter++;
        uint256 propertyId = propertyCounter;

        properties[propertyId] = Property({
            id: propertyId,
            name: name,
            location: location,
            value: value,
            owner: msg.sender
        });

        emit PropertyTokenized(propertyId, name, location, value, msg.sender);
    }

    /**
     * @dev Transfers ownership of a property to another verified user.
     * @param propertyId ID of the property to transfer
     * @param newOwner Address of the new owner
     */
    function transferProperty(uint256 propertyId, address newOwner) external onlyVerifiedUser(newOwner) onlyVerifiedUser(msg.sender) {
        Property storage property = properties[propertyId];
        require(property.owner == msg.sender, "Only the owner can transfer the property");

        property.owner = newOwner;

        emit PropertyTransferred(propertyId, msg.sender, newOwner);
    }

    /**
     * @dev Gets the details of a property.
     * @param propertyId ID of the property
     * @return name, location, value, owner
     */
    function getPropertyDetails(uint256 propertyId) external view returns (string memory name, string memory location, uint256 value, address owner) {
        Property storage property = properties[propertyId];
        return (property.name, property.location, property.value, property.owner);
    }
}
```

#### Description of the Smart Contract

This smart contract, `RealEstateTokenization`, allows for the tokenization and management of real estate properties on the Evire blockchain. It leverages the `EvireIdentityVerificationLibrary` to ensure that only verified users can participate in the tokenization and transfer of properties.

Key functionalities include:

* Allows verified users to tokenize new properties by providing property details like name, location, and value. Each property is assigned a unique ID.
* Enables the transfer of property ownership between verified users, ensuring compliance and security.
* Provides a method to retrieve the details of a specific property.

This contract demonstrates the integration and usage of the `EvireIdentityVerificationLibrary` to enhance security and regulatory compliance in real-world asset transactions.
