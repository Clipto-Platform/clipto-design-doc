## Migrations
This doc covers strategies used for contract data migrations in general and how can we migrate
data on the Clipto contracts.

### Current State
The contracts are not upgradable and there is no Proxy contracts, the main contract 
`CliptoExchange` stores all the state for the platform, while `CliptoToken` is the NFT contract
which gets cloned for each creator.

The CliptoExchange emits almost all of the required data with events which are indexed by a 
subgraph. So we have a datasource which is equally reliable to use to migrate from.

### Summary
- Ethereum’s strength is code immutability – but this makes it hard to upgrade smart contracts.
- The Logic-Data upgrade pattern is suitable if your state variables won’t change. Otherwise, prefer 
Zeppelin’s ‘unstructured storage’ Proxy-Logic pattern
- Use the migration pattern for worst-case scenario insurance
- Contract migration require pausability, thorough event logging, and an owner-only ‘pre-launch’ 
mode for transferring data.
- `In short, either we implement Upgradable Pattern or just go with Manual Migrations by reading
data from old contract and feeding it into the new contract (gas fees will be an issue).`

### Upgrade Patterns
1. Proxies and Implementation

This pattern relies on a proxy contract and an implementation contract(also called logic contract).
The proxy knows the implementation contract address and delegates all calls it receives to it.

```solidity
contract Proxy {
    address implementation;
    fallback() external payable {
        return implementation.delegatecall.value(msg.value)(msg.data);
    }
}
```

Since the proxy uses a delegate call into the implementation, it is as if it were running the 
implementation's code as its own. It modifies its own storage and balance, and preserves the
original `msg.sender` of the call. 

```
User ---- tx ---> Proxy ----------> Implementation_v0
                     |
                      ------------> Implementation_v1
                     |
                      ------------> Implementation_v2
```

2. Logic-Data Pattern

This pattern is suitable if you think you won’t need new state variables in future upgrades.

The pattern has the following properties:

 - The logic contract owns the data contract – only the logic contract can execute transactions that write to the data contract
 - The logic contract is `Pausable` – i.e. it inherits from the OpenZeppelin Pausable.sol contract, or implements identical functionality
 - Ownership of the data contract can be transferred to a new logic contract, from an authorized dApp admin address
 - The data contract’s functionality is limited to getters and setters for its state variables


 To Upgrade:
  - Deploy new logic contract
  - Pause the old logic contract
  - Change the ownership of the data contract to the new logic contract


Advantages of the Logic-Data Upgrade Pattern
 - Simple structure. Two contracts (or three with a proxy), and only one contract to be swapped out.
 - Simple flow of function calls between Logic and Data
 - Avoids all the pitfalls of delegatecall

Drawbacks
 - Logic contract doesn’t have a fixed address, unless you add an intermediate proxy contract (and gas costs)
 - Doesn’t allow addition of state variables in an upgrade


### Migration - Worst Case
Reasons for migrating our contract include significant upgrades to data and logic – for example, 
patching a critical bug in contract functionality, while also resetting user token balances after a 
hack.

Process:
 - Freeze the source contract
 - Recover the data to be migrated
 - Deploy a new destination contract in the pre-launch mode
 - Migrate the data from source to destination
 - Test the data at the migrated contract
 - Set the newly deployed contract to production mode
 - Update all relevant businesses – exchanges, wallets, other dApps, etc – with your new contract
  address

Recovering your contract data:
 - Public variables are easy to read
 - Array and private variables can be grabbed by calling `getStorageAt()` function.
 - Mappings would be hard
 - Emitted events will help recovering of all data

Writing to the destination contract:
 - Contract with pre-launch mode, calling batch transfer function, passing blobs of old data
 - Gas limits will catch up soon depending on the size of the data
 - Test the new storage


### References
- [Migrating Smart Contracts](http://ethdevs.com/upgrading-and-migrating-smart-contracts-on-ethereum/)
- [OpenZeppelin Upgradability](https://docs.openzeppelin.com/learn/upgrading-smart-contracts#how-upgrades-work)
- [Smart Contract Upgrades](https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/#upgrades-alternatives)
