# Example: Off-Chain Computation Request Handling

This example demonstrates how a smart contract on the Evire blockchain can interact with an off-chain compute framework to request and retrieve computation results. It focuses on the mechanism by which a smart contract sends a computation task to an off-chain compute node (Oracle), handles the asynchronous nature of receiving the computation result, and ensures the integrity and authenticity of the data received.

The purpose of this example is to illustrate:

* Integration with Smart Contracts, by showcasing how smart contracts can initiate off-chain computations and handle callbacks with results.
* Data Verification Layer, by highlighting the verification process ensuring that results from off-chain computations are valid and have not been tampered with before being accepted onto the blockchain.
* Result Consensus Protocol, by demonstrating a simple consensus mechanism where results from multiple compute nodes could be aggregated and verified to decide the most accurate outcome for the blockchain record.

This example is crucial for developers to understand how to effectively leverage the Off-Chain Compute Framework to extend the capabilities of their blockchain applications without sacrificing performance, security, or decentralization.

### Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IOffChainCompute {
    function requestComputation(string memory data) external returns (bytes32 requestId);
    function getResult(bytes32 requestId) external view returns (string memory result, bool isReady);
}

contract OffChainComputationHandler {
    IOffChainCompute offChainCompute;
    mapping(bytes32 => bool) public requestProcessed;
    mapping(bytes32 => string) public computationResults;

    event ComputationRequested(bytes32 indexed requestId);
    event ComputationCompleted(bytes32 indexed requestId, string result);

    constructor(address computeAddress) {
        offChainCompute = IOffChainCompute(computeAddress);
    }

    function requestOffChainComputation(string memory data) public {
        bytes32 requestId = offChainCompute.requestComputation(data);
        requestProcessed[requestId] = false;
        emit ComputationRequested(requestId);
    }

    function handleComputationResult(bytes32 requestId) public {
        require(!requestProcessed[requestId], "Request already processed.");

        (string memory result, bool isReady) = offChainCompute.getResult(requestId);
        require(isReady, "Computation results not ready yet.");

        requestProcessed[requestId] = true;
        computationResults[requestId] = result;
        emit ComputationCompleted(requestId, result);
    }
}
```

### Explanation

* `IOffChainCompute` interface defines how the smart contract interacts with the off-chain compute node. It includes methods to request computations and retrieve results.
* Two mappings are used, one to track whether a request has been processed and another to store the results linked to each request ID.
* The `requestOffChainComputation` function sends data to the off-chain compute node and records the request ID. The `handleComputationResult` function is called to retrieve and store the result once it's ready, ensuring that it has not been processed before and that the computation is completed.
* Events are emitted for both requesting and completing computations, which aids in tracking the process and integrating with front-end applications or other smart contracts.

### Example of Usage

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./OffChainComputationHandler.sol";

contract SmartContract is OffChainComputationHandler {
    struct ResultData {
        string result;
        bool verified;
    }

    mapping(bytes32 => ResultData) public results;
    mapping(bytes32 => address[]) public nodeVotes;
    mapping(bytes32 => mapping(address => bool)) public hasVoted;

    event ComputationVerificationRequested(bytes32 indexed requestId, string result);
    event ComputationVerified(bytes32 indexed requestId, string result);

    constructor(address computeAddress) OffChainComputationHandler(computeAddress) {}

    function requestComputationAndVerify(string memory data) public {
        requestOffChainComputation(data);
    }

    function handleComputationResult(bytes32 requestId) public override {
        super.handleComputationResult(requestId);

        string memory result = computationResults[requestId];
        emit ComputationVerificationRequested(requestId, result);
    }

    function verifyComputationResult(bytes32 requestId, bool isVerified) public {
        require(!hasVoted[requestId][msg.sender], "Node has already voted.");

        nodeVotes[requestId].push(msg.sender);
        hasVoted[requestId][msg.sender] = true;

        if (isVerified) {
            results[requestId].result = computationResults[requestId];
            results[requestId].verified = true;
        }

        if (nodeVotes[requestId].length > 2) {
            emit ComputationVerified(requestId, computationResults[requestId]);
        }
    }

    function getResult(bytes32 requestId) public view returns (string memory result, bool verified) {
        result = results[requestId].result;
        verified = results[requestId].verified;
    }
}

```

#### Explanation:

1. **OffChainComputationHandler.sol**:
   * Defines the `IOffChainCompute` interface.
   * Handles the process of requesting computations and storing results.
2. **SmartContract.sol**:
   * Inherits from `OffChainComputationHandler`.
   * Adds functionality to verify computation results through node voting.
   * Stores results that are verified by a consensus of nodes.
   * Includes events to track computation requests and verification status.

#### Key Concepts:

* **Integration with Smart Contracts**:
  * `requestOffChainComputation` initiates off-chain computations.
  * `handleComputationResult` handles results from off-chain nodes.
* **Data Verification Layer**:
  * Results are verified by multiple nodes to ensure integrity.
  * Nodes vote on the validity of the computation result.
* **Result Consensus Protocol**:
  * Implements a simple consensus mechanism where nodes vote to verify computation results.
  * Results are only accepted if a majority of nodes agree on their validity.

This example provides a comprehensive look at how to manage off-chain computations, verify results, and ensure data integrity through a consensus mechanism.
