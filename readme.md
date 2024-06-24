# Temple DAO

### Prize Pool

- Total Pool - $25,000
- H/M -  20k
- Low -2.5k
- Community Judging - 2.5k

- nSLOC: 998

- Starts: July 4, 2024 Noon UTC
- Ends: July 11, 2024 Noon UTC

## About the Project

TempleDAO is a DAO created to be a safe haven in a sea of risky assets, to facilitate long-term, stable wealth creation with a community-first mindset, and to elevate the overall DeFi experience for investors.
TempleDAO introduces a new cross-chain ERC20 token TempleGold which will be distributed to users through various auctions and Temple staking, as well as allocating a portion to the team. TempleGold inherits from LayerZero’s OFT standard to make cross-chain available while the token is only minted on Arbitrum chain.

Temple Gold is an equity claim on future airdrops. Airdrops are farmed token allocations from protocols like Ethena or Origami.
We introduce Temple Gold as a non-tradable claim on farmed token allocations. Using auction markets and staking contracts, users can get some emission of TGLD to claim farmed token allocations that temple creates in the future.
Users can claim allocations from specialized auctions called Spice auctions.

[TempleGold Documentation](https://github.com/TempleDAO/temple/tree/templegold/protocol/contracts/templegold/README.md)
[Website](https://templedao.link/)
[TempleGold Github branch](https://github.com/TempleDAO/temple/tree/templegold)

## Actors

```
Actors:
    Elevated Access: Gnosis multisig address of Temple DAO. Not set up yet but ideally a 3/4 multisig address. Elevated access controls staking and TGLD parameters. There is centralization risks if 3 addresses get compromised.
    Staker: Account staking Temple for TGLD rewards
    DAO Executor: Temple DAO governance executor contract. Controls spice auction configuration and parameters through governance.
    Distribution starter: A bot if set to non-zero address. Distributes TGLD for the next reward epoch
```

## Scope (contracts)

All Contracts in `contracts/templegold` are in scope.
```js
contracts/
└── templegold
    ├── AuctionBase.sol
    ├── DaiGoldAuction.sol
    ├── EpochLib.sol
    ├── SpiceAuction.sol
    ├── SpiceAuctionFactory.sol
    ├── TempleGold.sol
    ├── TempleGoldAdmin.sol
    ├── TempleGoldStaking.sol
    └── TempleTeleporter.sol
```

## Compatibilities

Please outline specific compatibilities of the protocol ie. blockchains (All EVM Compatible, specific chains). Please also include specific tokens, referencing standard contracts/interfaces when necessary, that are expected to function with the protocol. Include all whitelisted, or blacklisted tokens which are or are not supported. If the protocol is expected to function with **any** chain compatible tokens, please specify this.

```
Compatibilities:
  Blockchains:
      - Ethereum/Any EVM
  Tokens:
      - DAI
      - ENA
      - Temple
      - [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)
```

## Setup

Please outline specific steps/processes to be followed in order for an auditor to run the project off a local clone of the contest repo. Please again be detailed and thorough, including specific markdowned CLI commands and necessary .env adjustments.

Please also include steps needed to run appropriate tests included in scope.

Requirements:
- Yarn
- Node

Node:
```bash
# use node version > 18
# Example
nvm use 20.0.0
```

Setup:
```bash
# clone repository
git clone git@github.com:TempleDAO/temple.git
cd temple/protocol
# switch branch
git checkout templegold
# install dependencies
yarn install
```

Build:
```bash
# install forge dependencies
forge install

forge build
```

Tests:
```bash
# match contract
forge test --mc TempleGoldStaking
```

## Known Issues

Known Issues:
- Dai-Gold auction might not get started for last mint of gold tokens - 
Usually minting gold happens when the available amount is bigger than 1e4, which will configure the minimum distribution amount of DaiGoldAuction. However when it reaches the max supply, the minting amount can be smaller than minimum amount, thus DaiGoldAuction might not get started because the amount will be smaller than the minimum distribution amount.
For the last mint (leading to max supply), config.auctionMinimumDistributedGold will be updated to prevent this.

- In Temple Gold contract, setDistributionParams() can be executed with 0 value for a parameter eg. params.gnosis -
This is intentional to freely distribute minted TGLD to staking and dai gold auction only.

- Support for fee-on transfer tokens in spice auctions -
We do not expect to have spice tokens with fee on transfer. This is checked in contract nonetheless.

- Distribution starter is a trusted role

- Staking rewards for last epoch might not be distributed if remaining rewards are less than reward duration (dust amount) -
Dust amounts which are rewards less than reward duration for the last epoch will not be distributed. 
This is from L190 in `TempleGoldStaking.sol`.
```solidity
if (rewardAmount < rewardDuration ) { revert CommonEventsAndErrors.ExpectedNonZero(); }
```
We will redirect enough TGLD to staking for the last distribution.

- Current design of TempleGold prevents distribution of tokens -
With current TempleGold token design, tokens only can be transferred to/from whitelisted addresses. This prevents further distribution of TempleGold tokens where they could be listed on DEXes, put on lending pro- tocols as collateral and so on.
This is intended to make TGLD non-tradable.

- TempleGold incompatibility with some chains -
Because of PUSH0 not supported in `0.8.19` or lower versions of solidity compiler, TempleGold will be
incompatible with chains like Linea where it only supports solidity compiler 0.8.19 or lower.
