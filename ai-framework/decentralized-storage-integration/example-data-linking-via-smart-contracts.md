# Example: Data Linking via Smart Contracts

### **Step-by-Step** Implementation

1. **Smart Contract Definition**

Define the smart contract that will store hashes of the data.

```solidity
pragma solidity ^0.8.0;

contract DataLinker {
    struct DataReference {
        string dataHash;
        string description;
        uint timestamp;
    }

    mapping(uint => DataReference) public dataReferences;
    uint public dataCount;

    event DataLinked(uint indexed dataId, string dataHash, string description, uint timestamp);

    function linkData(string memory _dataHash, string memory _description) public {
        dataCount++;
        dataReferences[dataCount] = DataReference(_dataHash, _description, block.timestamp);
        emit DataLinked(dataCount, _dataHash, _description, block.timestamp);
    }

    function getData(uint _dataId) public view returns (string memory, string memory, uint) {
        DataReference memory dataRef = dataReferences[_dataId];
        return (dataRef.dataHash, dataRef.description, dataRef.timestamp);
    }
}
```

2. **Data Storage Off-Chain**

Store the actual data on a decentralized storage network such as IPFS (InterPlanetary File System). Upload the data to IPFS and retrieve the hash.

```sh
ipfs add path/to/datafile
```

This command will return a hash, for example: `QmXoYP4F2zYF3...`.

3. **Link Data to Smart Contract**

Use the hash obtained from IPFS to link the data to the smart contract.

```javascript
const Web3 = require('web3');
const web3 = new Web3('https://your-evire-node');

const contractABI = [ /* ABI from the compiled smart contract */ ];
const contractAddress = '0xYourContractAddress';
const dataLinker = new web3.eth.Contract(contractABI, contractAddress);

const dataHash = 'QmXoYP4F2zYF3...';
const description = 'Example data description';

dataLinker.methods.linkData(dataHash, description).send({ from: '0xYourAddress' })
    .on('receipt', function(receipt){
        console.log('Data linked:', receipt);
    });
```

4. **Retrieve Linked Data**

Fetch the linked data from the smart contract using its ID.

```javascript
dataLinker.methods.getData(1).call()
    .then(function(result) {
        console.log('Data Hash:', result[0]);
        console.log('Description:', result[1]);
        console.log('Timestamp:', result[2]);
    });
```

### Benefits

* Immutable References, with hashes stored on-chain serving as immutable references to the off-chain data, ensuring data integrity and security.
* Data Integrity, by using cryptographic hashes, any alteration in the actual data will result in a different hash, making it easy to detect tampering.
* Efficiency, by storing large data off-chain, reducing the storage burden on the blockchain, and improving performance and scalability.
* Flexibility, allowing for the integration of various decentralized storage solutions, adapting to different application needs.

### Considerations

* Data Privacy, by ensuring that sensitive data is adequately encrypted before uploading it to decentralized storage.
* Redundancy, through the utilization of multiple storage nodes and redundancy protocols to avoid single points of failure and enhance data availability.
* Cost, by being mindful of the transaction costs associated with storing and retrieving data references on the blockchain.

Implementing data linking via smart contracts on the Evire blockchain provides a robust solution for maintaining the integrity and accessibility of off-chain data. By leveraging decentralized storage networks, this approach combines the strengths of blockchain immutability with the scalability of off-chain data management,&#x20;
