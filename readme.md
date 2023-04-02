![RewardsDistributor](./img/aave_Reward_Distributor.png "aaveRewards")

# aave-v3-periphery > RewardsDistributor

being an abstract contract this contract is meant to be extended by other contracts that want to implement their own reward distribution mechanisms in the **aave protocol**

RewardsDistributor is an _abstract contract_ that manages multiple staking distributions with multiple rewards. This contract licensed under the BUSL-1.1 is intended to be used by other parent contract on the aave protocol as an accounting contract, tracking the distribution of rewards for a given set of decentralized non-custodial liquidity protocol provided by AAVE.

##### Prerequisites
This contract requires Solidity version 0.8.10 or above!.
& relies on the following external contracts:

1. IScaledBalanceToken from @aave/core-v3/contracts/interfaces/IScaledBalanceToken.sol
1. IERC20Detailed from @aave/core-v3/contracts/dependencies/openzeppelin/contracts/IERC20Detailed.sol
1. SafeCast from @aave/core-v3/contracts/dependencies/openzeppelin/contracts/SafeCast.sol
1. IRewardsDistributor from ./interfaces/IRewardsDistributor.sol
1. RewardsDataTypes from ./libraries/RewardsDataTypes.sol

## Storage Constants

- EMISSION_MANAGER: the address of the manager of incentives.
- Mappings
   - _assets: a mapping of rewarded asset addresses and their data (assetAddress => assetData).
   - _isRewardEnabled: a mapping of reward assets and their enabled status (rewardAddress => enabled).
- Arrays
   - _rewardsList: a list of reward asset addresses.
   - _assetsList: a list of asset addresses.

- Modifiers
   - onlyEmissionManager: This modifier restricts the execution of a function to the EMISSION_MANAGER address. If the caller is not the EMISSION_MANAGER, the function will revert with the error message ONLY_EMISSION_MANAGER.

- Functions
  /// @inheritdoc IRewardsDistributor

| Function Name | parameter(s) | returns |
| --- | --- | --- |
| constructor | address emissionManager | - |
| getRewardsData | address asset, address reward | uint256, uint256, uint256, uint256 |
| getAssetIndex | address asset, address reward | uint256, uint256 |
| getDistributionEnd | address asset, address reward | uint256 |
| getRewardsByAsset | address asset | address[] memory |\
| getRewardsList | - | address[] memory |
| getUserAssetIndex | three addresses | uint256 |
| getUserAccruedRewards | two addresses | uint256 |
| getUserRewards | an address array & two addresses | uint256 |
| getAllUserRewards | address array & an address | address array & uint256 |
| setDistributionEnd | two addresses | - |
| setEmissionPerSecond | an address, an address array & a uint88 array | - |

- Functions to configure the _assets for a specific emission

| Function Name | parameter(s) | returns |
| --- | --- | --- |
| _configureAssets | RewardsDataTypes.RewardsConfigInput[] | - |
|

   - constructor: The constructor of this contract takes an emissionManager address as a parameter, which is the address of the manager of incentives.

   - getRewardsData: This function returns information about the rewards for a given asset and reward. It takes two parameters: asset (the address of the asset being rewarded) and reward (the address of the reward token). It returns a tuple containing the following values:
      - index: the index of the reward in the rewards list.
      - emissionPerSecond: the amount of reward tokens distributed per second.
      - lastUpdateTimestamp: the timestamp of the last reward distribution update.
      - distributionEnd: the timestamp of the end of the reward distribution period.


   - getAssetIndex: this function returns the index of a given asset in the reward distribution. It takes in an asset address and a reward address as arguments, and returns two uint256 values
       - the index and the last time the distribution was updated.

   - getDistributionEnd: this function returns the timestamp when the distribution of a given asset reward will end. It takes in an asset address and a reward address as arguments, and returns a uint256 value representing the timestamp.

   - getRewardsByAsset: this function returns an array of address values representing the rewards available for a given asset. It takes in an asset address as an argument and returns an array of address values.

   - getRewardsList: this function returns an array of address values representing all the rewards available for all assets. It takes no arguments and returns an array of address values.

   - getUserAssetIndex: this function returns the index of a user's asset in the reward distribution for a given reward. It takes in the user's address, the asset address, and the reward address as arguments and returns a uint256 value representing the index.

   - getUserAccruedRewards function returns the total amount of a given reward that a user has accrued across all assets. It takes in a user address and a reward address as arguments and returns a uint256 value representing the total amount accrued.

   - getUserRewards: this function returns the amount of a given reward that a user has accrued across a specific set of assets. It takes in an array of assets addresses, a user address, and a reward address as arguments and returns a uint256 value representing the amount accrued.

   - getAllUserRewards: this function returns an array of rewards and the corresponding unclaimed amounts for a given user across a specific set of assets. It takes in an array of assets addresses and a user address as arguments, and returns two arrays - one containing the address values of the rewards, and the other containing the corresponding unclaimed amounts as uint256 values.

   - setDistributionEnd: this function allows the emission manager to update the distribution end timestamp for a specific reward and asset; it takes three parameters: asset, reward, and newDistributionEnd. It is used to update the distribution end time for a specific reward of an asset. The function can only be called by the emission manager. The old distribution end time is stored in the variable oldDistributionEnd, and the new distribution end time is stored in the rewards data structure for the given asset and reward. The function emits an event AssetConfigUpdated with the updated information.

   - setEmissionPerSecond: this function allows the emission manager to update the emission rate per second for one or more rewards on a specific asset, this function takes three parameters: the asset address, an array of rewards addresses, and an array of newEmissionsPerSecond values (both arrays of variable length). It first checks that the asset and reward exist and have valid configuration data, and then iterates over each reward, calling the _updateRewardData function and storing the updated index value. It then updates the emissionPerSecond value for the reward, emits an AssetConfigUpdated event with the updated emissionPerSecond and index values, and moves on to the next reward in the array.
   - _configureAssets: this is an internal, which means it can only be called within the contract itself. The purpose of this function is to configure the assets for a specific emission.
      1. The function takes an array of RewardsDataTypes.RewardsConfigInput struct as its input, which contains the configuration for each asset, such as the asset address, reward address, emission per second, and distribution end.
      1. The function iterates over the array of input configurations and checks if the asset has already been initialized before. If not, it adds the asset to the list of assets _assetsList.
      1. It then retrieves the decimals of the asset using the `IERC20Detailed` interface and sets it to the asset configuration.
      1. If the reward configuration's lastUpdateTimestamp is zero it adds the reward address to the asset's available rewards list and increments the count of available rewards for that asset
      1. If the reward address is still not enabled, it adds the reward address to the global rewards list _rewardsList.
      1. afterwhich it updates the reward data by calling the internal _updateRewardData function and sets the emission per second and distribution end for the reward configuration.
      1. This function is called by the configureAssets function during the initialization of the rewards program.
      1. Finally, it emits an _AssetConfigUpdated_ event with the old and new values of the emission per second and distribution end.

   - _updateRewardData function takes in the storage pointer to the distribution reward config, the current total supply of the asset, and the asset unit. It calculates the new distribution index for the reward and returns it along with a boolean value indicating whether the index was updated.

   - _updateUserData function takes in the storage pointer to the distribution reward config, the address of the user, the user's balance of the asset, the new index of the asset distribution, and the asset unit. It calculates the rewards accrued by the user since the last update and returns it along with a boolean value indicating whether the user's data was updated.

      - two functions above are internal functions are used in updating the state of the distribution for the specified reward and specific user
      - both functions update the lastUpdateTimestamp of the reward data to the current block timestamp. The reward data also contains information about the emissionPerSecond, distributionEnd, and the availableRewards. The function _updateRewardData updates the rewardData index and the function _updateUserData updates the user's index and accrued rewards.


These functions are related to updating the state of the distribution for a specific user and asset.

   - _updateRewardData: this function takes a storage pointer to a RewardData struct, the current total supply of the asset, and the unit of the asset (i.e., 10**decimals), and returns the new distribution index and a boolean indicating whether the index was updated. It first calculates the new index based on the current total supply and the reward rate and stores it in newIndex. If newIndex is different from the old index stored in rewardData, it updates the index and the last update timestamp. If the index is not updated, it only updates the last update timestamp.

   - _updateUserData: this function takes a storage pointer to a RewardData struct, a user address, the user balance of the asset, the new index of the asset distribution, and the unit of the asset, and returns the rewards accrued since the last update and a boolean indicating whether the user data was updated. If the user index is different from the new index, it updates the user index and calculates the rewards accrued based on the difference between the new and old indexes and the user balance of the asset. It then adds the rewards accrued to the user's accrued rewards. If the user data is not updated, it does not calculate any rewards.

   - _updateData: this function takes an asset address, a user address, the user balance of the asset, and the total supply of the asset, and updates the state of the distribution for all the rewards available for the asset. It first gets the unit of the asset and checks if there are any available rewards for the asset. If there are no rewards available, it returns. Otherwise, it iterates over all the available rewards and calls _updateRewardData and _updateUserData for each reward. If either the reward data or the user data is updated, it emits an Accrued event.

   - the _updateDataMultiple function takes a user address and an array of UserAssetBalance structs, each containing the user balance and total supply of an asset, and updates the state of the distribution for each asset using _updateData.

##### The following are internal functions used by the contract to calculate and update the accrued and unclaimed rewards for users based on their asset balances and the available rewards in the system.

2. The _updateData function iterates over all the available rewards for a specific asset and updates the reward data and user data for each one. This function is called whenever a user's asset balance changes or when new rewards are added to the system.

2. The _updateDataMultiple function is called by the contract's external functions to update the reward data and user data for multiple assets at once.

2. The _getUserReward function calculates the accrued but unclaimed rewards for a specific user based on their asset balances and the available rewards in the system. It calls the _getPendingRewards function to calculate the pending rewards for each asset and adds them to the user's accrued rewards.

2. The _getPendingRewards function calculates the pending (not yet accrued) rewards for a specific user and asset based on their balance and the available rewards in the system.

2. The _getRewards function is an internal utility function that calculates the rewards for a user based on their balance, the current index of the distribution, their stored index, and the asset unit. It's used by _getPendingRewards to calculate the pending rewards for a specific user and asset.

2. _getUserAssetBalances function retrieves the balance and total supply of a list of assets for a specified user. The _getAssetIndex function calculates the next value of an index for a specific distribution reward, taking into account the total supply and emission rate of the asset being rewarded.

2. _getRewards function calculates the rewards for a user based on their asset balance and the difference between their staking moment and the current moment. The _getPendingRewards function calculates the rewards that have not yet been accrued by a user since their last interaction with the system.

2. _getUserReward function combines the results of the previous functions to calculate the total accrued but unclaimed rewards for a user across all incentivized assets.

this abstarct contract also includes two external functions, getAssetDecimals and getEmissionManager, that allow external contracts to retrieve information about the assets and the emission manager used by the rewards distribution system.


