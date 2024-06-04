# Example: AI-Powered Predictive Analytics dApp

1. **Smart Contract for Task Submission**

First, we need a smart contract that allows users to submit AI computation tasks.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AIComputeTask {
    uint public taskCount = 0;

    struct Task {
        uint id;
        address requester;
        string dataHash;
        string modelType;
        uint reward;
        bool completed;
        string resultHash;
    }

    mapping(uint => Task) public tasks;

    event TaskCreated(uint id, address requester, string dataHash, string modelType, uint reward);
    event TaskCompleted(uint id, string resultHash);

    function createTask(string memory dataHash, string memory modelType) public payable {
        require(msg.value > 0, "Reward must be greater than zero");

        taskCount++;
        tasks[taskCount] = Task(taskCount, msg.sender, dataHash, modelType, msg.value, false, "");

        emit TaskCreated(taskCount, msg.sender, dataHash, modelType, msg.value);
    }

    function completeTask(uint taskId, string memory resultHash) public {
        Task storage task = tasks[taskId];
        require(task.id == taskId, "Task does not exist");
        require(!task.completed, "Task already completed");

        task.completed = true;
        task.resultHash = resultHash;

        payable(msg.sender).transfer(task.reward);

        emit TaskCompleted(taskId, resultHash);
    }
}

```

2. **Off-Chain Compute Node Setup**

Next, we configure the off-chain compute nodes to listen for new tasks, perform the computations, and submit the results back to the smart contract.

```python
import requests
from web3 import Web3

# Initialize web3
w3 = Web3(Web3.HTTPProvider('https://your-eth-node-url'))

# Contract ABI and address
contract_abi = '[...]'  # Replace with actual ABI
contract_address = '0xYourContractAddress'

# Load the contract
contract = w3.eth.contract(address=contract_address, abi=contract_abi)

# Function to listen for new tasks
def listen_for_tasks():
    event_filter = contract.events.TaskCreated.createFilter(fromBlock='latest')
    while True:
        for event in event_filter.get_new_entries():
            handle_task_event(event)

def handle_task_event(event):
    task_id = event['args']['id']
    data_hash = event['args']['dataHash']
    model_type = event['args']['modelType']

    # Fetch data and perform computation (mock example)
    result = perform_ai_computation(data_hash, model_type)

    # Submit result back to the contract
    submit_result(task_id, result)

def perform_ai_computation(data_hash, model_type):
    # Simulate AI computation (replace with actual model execution)
    result_hash = "computed_result_hash_based_on_" + data_hash
    return result_hash

def submit_result(task_id, result_hash):
    tx = contract.functions.completeTask(task_id, result_hash).buildTransaction({
        'from': w3.eth.defaultAccount,
        'nonce': w3.eth.getTransactionCount(w3.eth.defaultAccount)
    })
    signed_tx = w3.eth.account.signTransaction(tx, private_key='your_private_key')
    w3.eth.sendRawTransaction(signed_tx.rawTransaction)

if __name__ == "__main__":
    listen_for_tasks()

```

3. **Load Balancing and Incentive Mechanism**

We ensure that tasks are efficiently distributed among compute nodes and that nodes are incentivized based on performance.

```python
import random
from queue import PriorityQueue

# Priority queue to manage compute node load
compute_nodes = PriorityQueue()

# Register compute nodes
def register_node(node_address, compute_power):
    compute_nodes.put((compute_power, node_address))

# Assign task to the node with highest compute power available
def assign_task_to_node(task_id, data_hash, model_type):
    if not compute_nodes.empty():
        compute_power, node_address = compute_nodes.get()
        # Send task details to the selected node
        response = requests.post(f"http://{node_address}/perform_task", json={
            'task_id': task_id,
            'data_hash': data_hash,
            'model_type': model_type
        })
        # Reinsert node into queue with updated compute power (mock logic)
        new_compute_power = max(0, compute_power - random.randint(1, 10))  # Ensure compute power doesn't go negative
        compute_nodes.put((new_compute_power, node_address))
        return response.json()
    else:
        print("No compute nodes available")
        return None

```

4. **Security Audits and Compliance**

Regular audits and compliance checks are crucial to maintain trust and security in the framework.

```python
import hashlib

def perform_security_audit(task_id, data_hash, result_hash):
    # Verify data integrity (mock logic)
    expected_result_hash = hashlib.sha256(data_hash.encode()).hexdigest()
    if expected_result_hash == result_hash:
        print(f"Task {task_id} passed security audit")
        return True
    else:
        print(f"Task {task_id} failed security audit")
        return False

# Example usage
task_id = 1
data_hash = "example_data_hash"
result_hash = perform_ai_computation(data_hash, "example_model")
if perform_security_audit(task_id, data_hash, result_hash):
    submit_result(task_id, result_hash)

```

This example provides a full lifecycle from task creation, off-chain computation, result submission, load balancing, and security auditing within the Evire Off-Chain Compute Framework. By leveraging smart contracts, decentralized compute nodes, and robust security measures, developers can efficiently handle complex AI tasks off-chain while maintaining blockchain integrity and trust .
