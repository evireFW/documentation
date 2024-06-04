# Example: Evire Player Stats Library

In competitive gaming, tracking player statistics such as scores, rankings, and achievements is essential. It provides a basis for matchmaking, leaderboards and overall enhancement of competitive play. Evire incorporates player statistics tracking within its smart contract libraries, enabling these metrics to be stored securely on the blockchain. This not only ensures that player data cannot be falsified but also makes the data transparent and accessible, fostering a competitive environment where players can see how they stack up against others globally.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library EvirePlayerStats {
    struct PlayerStats {
        uint256 gamesPlayed;
        uint256 gamesWon;
        uint256 totalScore;
        uint256 highestScore;
        uint256 rank;
        mapping(string => uint256) achievements;
    }

    function initialize(PlayerStats storage self) internal {
        self.gamesPlayed = 0;
        self.gamesWon = 0;
        self.totalScore = 0;
        self.highestScore = 0;
        self.rank = 0;
    }

    function recordGame(PlayerStats storage self, uint256 score, bool isWin) internal {
        self.gamesPlayed += 1;
        self.totalScore += score;
        if (isWin) {
            self.gamesWon += 1;
        }
        if (score > self.highestScore) {
            self.highestScore = score;
        }
        // Add logic to update rank based on the game's ranking algorithm
    }

    function addAchievement(PlayerStats storage self, string memory achievement, uint256 value) internal {
        self.achievements[achievement] = value;
    }

    function getPlayerStats(PlayerStats storage self) internal view returns (
        uint256 gamesPlayed,
        uint256 gamesWon,
        uint256 totalScore,
        uint256 highestScore,
        uint256 rank
    ) {
        return (
            self.gamesPlayed,
            self.gamesWon,
            self.totalScore,
            self.highestScore,
            self.rank
        );
    }

    function getAchievement(PlayerStats storage self, string memory achievement) internal view returns (uint256) {
        return self.achievements[achievement];
    }
}

```

This `EvirePlayerStats` library provides essential functions to initialize player statistics, record game results, add achievements, and retrieve player statistics and achievements. By incorporating this library into a smart contract, developers can create secure, transparent, and immutable records of player statistics, enhancing the competitive gaming experience on the Evire blockchain platform.

### Example of usage:

This dApp demonstrates how to use the `EvirePlayerStats` library to track and manage player statistics in a gaming context. The dApp includes functions for player registration, recording game results, adding achievements, and viewing player statistics.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./GamingFW/EvirePlayerStats.sol";

contract PlayerStatsDApp {
    using EvirePlayerStats for EvirePlayerStats.PlayerStats;

    struct Player {
        address playerAddress;
        EvirePlayerStats.PlayerStats stats;
    }

    mapping(address => Player) public players;
    address[] public playerAddresses;

    event PlayerRegistered(address indexed playerAddress);
    event GameRecorded(address indexed playerAddress, uint256 score, bool isWin);
    event AchievementAdded(address indexed playerAddress, string achievement, uint256 value);

    function registerPlayer() external {
        require(players[msg.sender].playerAddress == address(0), "Player already registered");

        players[msg.sender].playerAddress = msg.sender;
        players[msg.sender].stats.initialize();
        playerAddresses.push(msg.sender);

        emit PlayerRegistered(msg.sender);
    }

    function recordGame(uint256 score, bool isWin) external {
        require(players[msg.sender].playerAddress != address(0), "Player not registered");

        players[msg.sender].stats.recordGame(score, isWin);

        emit GameRecorded(msg.sender, score, isWin);
    }

    function addAchievement(string calldata achievement, uint256 value) external {
        require(players[msg.sender].playerAddress != address(0), "Player not registered");

        players[msg.sender].stats.addAchievement(achievement, value);

        emit AchievementAdded(msg.sender, achievement, value);
    }

    function getPlayerStats(address playerAddress) external view returns (
        uint256 gamesPlayed,
        uint256 gamesWon,
        uint256 totalScore,
        uint256 highestScore, uint256 rank) {require(players[playerAddress].playerAddress != address(0), "Player not registered");
        return players[playerAddress].stats.getPlayerStats();
    }
    
    function getAchievement(address playerAddress, string calldata achievement) external view returns (uint256) {
        require(players[playerAddress].playerAddress != address(0), "Player not registered");
    
        return players[playerAddress].stats.getAchievement(achievement);
    }
    
    function getAllPlayers() external view returns (address[] memory) {
        return playerAddresses;
    }
}
```

In this `PlayerStatsDApp` contract, the following functionalities are provided:

1. **Player Registration**:
   * Players can register themselves using the `registerPlayer` function. This initializes their player statistics and adds them to the player list.
2. **Recording Game Results**:
   * The `recordGame` function allows registered players to record the results of their games, updating their statistics accordingly.
3. **Adding Achievements**:
   * Players can add achievements using the `addAchievement` function, which records the achievement and its value in their statistics.
4. **Viewing Player Statistics**:
   * The `getPlayerStats` function returns the statistics of a specified player.
   * The `getAchievement` function retrieves the value of a specified achievement for a player.
5. **Listing All Players**:
   * The `getAllPlayers` function returns a list of all registered player addresses.

This example demonstrates how to integrate the `EvirePlayerStats` library within a dApp, allowing for the management of player statistics on the Evire blockchain.
