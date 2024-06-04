# Example: Physical Infrastructure Management Library

This library is designed to handle operations involved in infrastructure management, ensuring data integrity and facilitating integration with IoT data sources for real-time monitoring and decision-making. The modular and extensible nature of the library allows developers to customize and scale it according to their specific project requirements.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/// @notice This library provides reusable smart contract modules for managing physical infrastructure on the blockchain.
library InfrastructureManagement {

    struct Asset {
        string name;
        uint256 id;
        address owner;
        uint256 createdAt;
        uint256 updatedAt;
        bool active;
    }

    struct MaintenanceRecord {
        uint256 assetId;
        uint256 maintenanceDate;
        string description;
        address performedBy;
        bool completed;
    }

    struct ResourceAllocation {
        uint256 assetId;
        uint256 resourceId;
        uint256 quantity;
        uint256 allocatedAt;
        bool active;
    }

    event AssetCreated(uint256 assetId, string name, address owner);
    event AssetUpdated(uint256 assetId, string name, address owner, bool active);
    event MaintenanceScheduled(uint256 assetId, uint256 maintenanceDate, string description, address performedBy);
    event MaintenanceCompleted(uint256 assetId, uint256 maintenanceDate, address performedBy);
    event ResourceAllocated(uint256 assetId, uint256 resourceId, uint256 quantity, uint256 allocatedAt);
    event ResourceDeallocated(uint256 assetId, uint256 resourceId, uint256 quantity, uint256 deallocatedAt);

    function createAsset(mapping(uint256 => Asset) storage assets, uint256 assetId, string memory name, address owner) public {
        require(assets[assetId].id == 0, "Asset already exists");

        assets[assetId] = Asset({
            name: name,
            id: assetId,
            owner: owner,
            createdAt: block.timestamp,
            updatedAt: block.timestamp,
            active: true
        });

        emit AssetCreated(assetId, name, owner);
    }

    function updateAsset(mapping(uint256 => Asset) storage assets, uint256 assetId, string memory name, address owner, bool active) public {
        require(assets[assetId].id != 0, "Asset does not exist");

        Asset storage asset = assets[assetId];
        asset.name = name;
        asset.owner = owner;
        asset.updatedAt = block.timestamp;
        asset.active = active;

        emit AssetUpdated(assetId, name, owner, active);
    }

    function scheduleMaintenance(mapping(uint256 => MaintenanceRecord[]) storage maintenanceRecords, uint256 assetId, uint256 maintenanceDate, string memory description, address performedBy) public {
        maintenanceRecords[assetId].push(MaintenanceRecord({
            assetId: assetId,
            maintenanceDate: maintenanceDate,
            description: description,
            performedBy: performedBy,
            completed: false
        }));

        emit MaintenanceScheduled(assetId, maintenanceDate, description, performedBy);
    }

    function completeMaintenance(mapping(uint256 => MaintenanceRecord[]) storage maintenanceRecords, uint256 assetId, uint256 maintenanceDate, address performedBy) public {
        MaintenanceRecord[] storage records = maintenanceRecords[assetId];
        for (uint256 i = 0; i < records.length; i++) {
            if (records[i].maintenanceDate == maintenanceDate && records[i].performedBy == performedBy && !records[i].completed) {
                records[i].completed = true;
                emit MaintenanceCompleted(assetId, maintenanceDate, performedBy);
                break;
            }
        }
    }

    function allocateResource(mapping(uint256 => ResourceAllocation[]) storage resourceAllocations, uint256 assetId, uint256 resourceId, uint256 quantity) public {
        resourceAllocations[assetId].push(ResourceAllocation({
            assetId: assetId,
            resourceId: resourceId,
            quantity: quantity,
            allocatedAt: block.timestamp,
            active: true
        }));

        emit ResourceAllocated(assetId, resourceId, quantity, block.timestamp);
    }

    function deallocateResource(mapping(uint256 => ResourceAllocation[]) storage resourceAllocations, uint256 assetId, uint256 resourceId, uint256 quantity) public {
        ResourceAllocation[] storage allocations = resourceAllocations[assetId];
        for (uint256 i = 0; i < allocations.length; i++) {
            if (allocations[i].resourceId == resourceId && allocations[i].quantity == quantity && allocations[i].active) {
                allocations[i].active = false;
                emit ResourceDeallocated(assetId, resourceId, quantity, block.timestamp);
                break;
            }
        }
    }
}
```

#### Explanation:

This Solidity library, `InfrastructureManagement`, provides essential functionalities to manage physical infrastructure assets, including:

* Functions to create and update assets with events for logging changes.
* Functions to schedule and complete maintenance activities, recording relevant details and emitting events.
* Functions to allocate and deallocate resources to assets, ensuring efficient resource management with event logging.

### Example of Usage:

This smart contract includes functions for asset creation, updating, scheduling maintenance, completing maintenance and resource allocation. It also incorporates access control mechanisms, detailed events and error handling for a robust and secure implementation.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./DePINFW/InfrastructureManagement.sol";

/// @title Physical Infrastructure Management Smart Contract
/// @notice This contract manages physical infrastructure assets, including their lifecycle, maintenance, and resource allocation.
contract PhysicalInfrastructureManager {
    using InfrastructureManagement for mapping(uint256 => InfrastructureManagement.Asset);
    using InfrastructureManagement for mapping(uint256 => InfrastructureManagement.MaintenanceRecord[]);
    using InfrastructureManagement for mapping(uint256 => InfrastructureManagement.ResourceAllocation[]);

    mapping(uint256 => InfrastructureManagement.Asset) private assets;
    mapping(uint256 => InfrastructureManagement.MaintenanceRecord[]) private maintenanceRecords;
    mapping(uint256 => InfrastructureManagement.ResourceAllocation[]) private resourceAllocations;

    address public admin;

    event AdminChanged(address indexed previousAdmin, address indexed newAdmin);
    event UnauthorizedAccess(address indexed caller);

    modifier onlyAdmin() {
        if (msg.sender != admin) {
            emit UnauthorizedAccess(msg.sender);
            revert("Caller is not the admin");
        }
        _;
    }

    constructor() {
        admin = msg.sender;
        emit AdminChanged(address(0), msg.sender);
    }

    function changeAdmin(address newAdmin) public onlyAdmin {
        require(newAdmin != address(0), "New admin is the zero address");
        emit AdminChanged(admin, newAdmin);
        admin = newAdmin;
    }

    function createAsset(uint256 assetId, string memory name, address owner) public onlyAdmin {
        assets.createAsset(assetId, name, owner);
    }

    function updateAsset(uint256 assetId, string memory name, address owner, bool active) public onlyAdmin {
        assets.updateAsset(assetId, name, owner, active);
    }

    function getAsset(uint256 assetId) public view returns (InfrastructureManagement.Asset memory) {
        return assets[assetId];
    }

    function scheduleMaintenance(uint256 assetId, uint256 maintenanceDate, string memory description, address performedBy) public onlyAdmin {
        maintenanceRecords.scheduleMaintenance(assetId, maintenanceDate, description, performedBy);
    }

    function completeMaintenance(uint256 assetId, uint256 maintenanceDate, address performedBy) public onlyAdmin {
        maintenanceRecords.completeMaintenance(assetId, maintenanceDate, performedBy);
    }

    function getMaintenanceRecords(uint256 assetId) public view returns (InfrastructureManagement.MaintenanceRecord[] memory) {
        return maintenanceRecords[assetId];
    }

    function allocateResource(uint256 assetId, uint256 resourceId, uint256 quantity) public onlyAdmin {
        resourceAllocations.allocateResource(assetId, resourceId, quantity);
    }

    function deallocateResource(uint256 assetId, uint256 resourceId, uint256 quantity) public onlyAdmin {
        resourceAllocations.deallocateResource(assetId, resourceId, quantity);
    }

    function getResourceAllocations(uint256 assetId) public view returns (InfrastructureManagement.ResourceAllocation[] memory) {
        return resourceAllocations[assetId];
    }

    function getAssetsByOwner(address owner) public view returns (InfrastructureManagement.Asset[] memory) {
        uint256 count;
        for (uint256 i = 0; i < 1e6; i++) {
            if (assets[i].owner == owner) {
                count++;
            }
        }

        InfrastructureManagement.Asset[] memory result = new InfrastructureManagement.Asset[](count);
        uint256 index;
        for (uint256 i = 0; i < 1e6; i++) {
            if (assets[i].owner == owner) {
                result[index] = assets[i];
                index++;
            }
        }

        return result;
    }

    function getActiveAssets() public view returns (InfrastructureManagement.Asset[] memory) {
        uint256 count;
        for (uint256 i = 0; i < 1e6; i++) {
            if (assets[i].active) {
                count++;
            }
        }

        InfrastructureManagement.Asset[] memory result = new InfrastructureManagement.Asset[](count);
        uint256 index;
        for (uint256 i = 0; i < 1e6; i++) {
            if (assets[i].active) {
                result[index] = assets[i];
                index++;
            }
        }

        return result;
    }
}
```

#### Explanation:

* The `InfrastructureManagement` library is imported and used within the contract to handle asset management, maintenance tracking, and resource allocation.
* The contract includes an `admin` address with functions restricted to the admin using the `onlyAdmin` modifier.
* An `UnauthorizedAccess` event is emitted if a non-admin attempts to perform restricted actions.
* Functions for creating, updating assets, scheduling, and completing maintenance, as well as allocating and deallocating resources, are restricted to the admin.
* Functions for retrieving assets, maintenance records, and resource allocations are available to all users.
* Additional helper functions allow users to fetch assets by owner and list active assets.
* Detailed events are emitted for administrative changes and unauthorized access attempts.
* Proper error handling is implemented using `revert` to ensure the contract behaves correctly in case of unauthorized access or invalid operations.

This smart contract demonstrates a secure approach to managing physical infrastructure assets on the blockchain, leveraging the reusable components provided by the `InfrastructureManagement` library.
