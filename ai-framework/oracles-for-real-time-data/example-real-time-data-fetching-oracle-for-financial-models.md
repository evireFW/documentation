# Example: Real-Time Data Fetching Oracle for Financial Models

This example demonstrates a Solidity smart contract that interacts with an oracle to fetch real-time market data for use in financial models on the Evire blockchain. The contract is designed to request and receive foreign exchange rate data, which could be crucial for financial applications such as trading bots, risk assessment tools, and dynamic pricing models.

The purpose of this example is to illustrate how to implement an oracle within a smart contract to securely and efficiently integrate real-time data into blockchain-based applications. By fetching up-to-date foreign exchange rates, the contract can make informed decisions based on current market conditions, enhancing its functionality and reliability.

**Example:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@chainlink/contracts/src/v0.8/ChainlinkClient.sol";

contract ForexOracle is ChainlinkClient {
    using Chainlink for Chainlink.Request;

    uint256 private fee;
    bytes32 private jobId;

    // Mapping to store the request data
    mapping(bytes32 => string) private requestCurrencyPairs;

    // Initialize the Chainlink token and oracle address
    constructor() {
        setPublicChainlinkToken();
        oracle = 0xOracleAddress; // Replace with the actual oracle address
        jobId = "jobId"; // Replace with the actual job ID
        fee = 0.1 * 10 ** 18; // 0.1 LINK (Varies by network and job)
    }

    // Function to request data from the oracle
    function requestData(string memory _currencyPair) public returns (bytes32 requestId) {
        Chainlink.Request memory request = buildChainlinkRequest(jobId, address(this), this.fulfill.selector);
        request.add("currencyPair", _currencyPair);
        requestId = sendChainlinkRequestTo(oracle, request, fee);
        requestCurrencyPairs[requestId] = _currencyPair;
        return requestId;
    }

    // Fulfill function to receive the data
    function fulfill(bytes32 _requestId, uint256 _rate) public recordChainlinkFulfillment(_requestId) {
        string memory currencyPair = requestCurrencyPairs[_requestId];
        // Process the received rate
        // e.g., store it or emit an event
    }
}
```

In this example, the `ForexOracle` contract uses Chainlink's `ChainlinkClient` to interact with the Chainlink network. Here’s a breakdown of the key components:

1. This contract inherits from `ChainlinkClient` to leverage Chainlink’s functionality.
2. Initializes the Chainlink token and sets the oracle and job ID.
3. Builds a Chainlink request, specifies the currency pair, sends the request to the Chainlink oracle, and maps the request ID to the currency pair.
4. The `fulfill` function is a callback that the Chainlink oracle calls to deliver the requested data. It requires the request ID and the rate as parameters. The function ensures that the caller is the authorized oracle, retrieves the currency pair associated with the request ID, and processes the received rate, such as storing it or emitting an event to notify other components.
5. This callback function is called by the Chainlink oracle to return the requested data. It processes the received rate and can store it or emit an event.

Below is a comprehensive example of a complex smart contract that integrates with the `ForexOracle.sol` library. This smart contract facilitates decentralized trading of foreign exchange rates and includes functionalities for placing trades, managing user balances, and ensuring compliance with forex rate updates.

### Example of Usage:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@chainlink/contracts/src/v0.8/ChainlinkClient.sol";
import "./ForexOracle.sol";

contract ForexTrading is ForexOracle {
    using Chainlink for Chainlink.Request;

    struct Trade {
        address trader;
        string currencyPair;
        uint256 amount;
        bool isBuy;
        uint256 rate;
        uint256 timestamp;
        bool isFulfilled;
    }

    mapping(address => uint256) public balances;
    mapping(bytes32 => Trade) public trades;
    bytes32[] public tradeIds;

    event TradePlaced(bytes32 indexed tradeId, address indexed trader, string currencyPair, uint256 amount, bool isBuy);
    event TradeFulfilled(bytes32 indexed tradeId, uint256 rate, uint256 timestamp);

    constructor() ForexOracle() {}

    function deposit() public payable {
        require(msg.value > 0, "Deposit amount must be greater than zero");
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) public {
        require(amount <= balances[msg.sender], "Insufficient balance");
        payable(msg.sender).transfer(amount);
        balances[msg.sender] -= amount;
    }

    function placeTrade(string memory _currencyPair, uint256 _amount, bool _isBuy) public returns (bytes32) {
        require(_amount > 0, "Trade amount must be greater than zero");
        require(_amount <= balances[msg.sender], "Insufficient balance");

        balances[msg.sender] -= _amount;

        bytes32 requestId = requestData(_currencyPair);
        Trade memory newTrade = Trade({
            trader: msg.sender,
            currencyPair: _currencyPair,
            amount: _amount,
            isBuy: _isBuy,
            rate: 0,
            timestamp: block.timestamp,
            isFulfilled: false
        });

        trades[requestId] = newTrade;
        tradeIds.push(requestId);

        emit TradePlaced(requestId, msg.sender, _currencyPair, _amount, _isBuy);

        return requestId;
    }

    function fulfill(bytes32 _requestId, uint256 _rate) public override recordChainlinkFulfillment(_requestId) {
        Trade storage trade = trades[_requestId];
        require(!trade.isFulfilled, "Trade already fulfilled");

        trade.rate = _rate;
        trade.isFulfilled = true;
        
        if (trade.isBuy) {
            balances[trade.trader] += trade.amount * _rate / 1e18;
        } else {
            balances[trade.trader] += trade.amount * (1e18 - _rate) / 1e18;
        }

        emit TradeFulfilled(_requestId, _rate, block.timestamp);
    }

    function getTradeDetails(bytes32 _tradeId) public view returns (Trade memory) {
        return trades[_tradeId];
    }

    function getAllTradeIds() public view returns (bytes32[] memory) {
        return tradeIds;
    }
}
```

#### Key Features:

1. Users can deposit ETH into their account and withdraw their balance.
2. Users can place a trade by specifying the currency pair, amount, and whether it's a buy or sell trade.
3. This function gets called when the oracle returns the exchange rate. It calculates the new balance based on the trade details and the retrieved rate.
4. The contract manages trades using a mapping from request IDs to trade details, allowing users to query their trade details.

This contract is designed to work seamlessly with the `ForexOracle.sol` library, utilizing the Chainlink Oracle to fetch real-time forex rates and managing user balances and trades efficiently.
