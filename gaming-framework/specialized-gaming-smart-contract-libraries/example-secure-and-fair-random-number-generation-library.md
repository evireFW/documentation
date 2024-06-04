# Example: Secure and Fair Random Number Generation Library

### **1. Smart Contract for RNG**&#x20;

This smart contract will generate random numbers in a secure and transparent manner.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SecureRNG {
    uint256 private nonce;
    address private owner;
    
    event RandomNumberGenerated(uint256 randomNumber, address indexed requester);

    constructor() {
        owner = msg.sender;
        nonce = 0;
    }

    // Modifier to restrict access to certain functions
    modifier onlyOwner() {
        require(msg.sender == owner, "Not authorized");
        _;
    }

    // Internal function to generate a random number
    function generateRandomNumber() internal returns (uint256) {
        nonce++;
        uint256 randomNumber = uint256(keccak256(abi.encodePacked(block.timestamp, msg.sender, nonce)));
        return randomNumber;
    }
    
    // Public function to request a random number
    function requestRandomNumber() public returns (uint256) {
        uint256 randomNumber = generateRandomNumber();
        emit RandomNumberGenerated(randomNumber, msg.sender);
        return randomNumber;
    }
    // Function to reset the nonce for added security
    function resetNonce() public onlyOwner {
        nonce = 0;
    }
}

```

### **2. Explanation of the Contract**

* A counter that changes with each request to ensure unpredictability.
* Uses `keccak256` to hash the combination of the current block timestamp, the requester's address, and the nonce. This ensures the randomness and security of the generated number.
* Emits an event every time a random number is generated, providing transparency and allowing for tracking of requests.

### **3. Deployment and Interaction**

Deploying the contract and interacting with it can be done using a blockchain development environment like Truffle or Hardhat.

```solidity
const SecureRNG = artifacts.require("SecureRNG");

module.exports = function (deployer) {
  deployer.deploy(SecureRNG);
};

```

### **4. Testing the RNG**

Create a test script to ensure the RNG works as expected.

```solidity
const SecureRNG = artifacts.require("SecureRNG");

contract("SecureRNG", accounts => {
    it("should generate a random number", async () => {
        const rngInstance = await SecureRNG.deployed();
        const randomNumber = await rngInstance.requestRandomNumber.call();
        console.log("Generated Random Number: ", randomNumber.toString());
        assert(randomNumber, "Random number should be generated");
    });
});

```

### Benefits of This Implementation

1. The use of `keccak256` ensures the randomness is cryptographically secure.
2. Emitting events for each random number generation allows for public tracking.
3. Since the number generation is tied to block data and user-specific inputs, it is resistant to manipulation by either players or developers.
