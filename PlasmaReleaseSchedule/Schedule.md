# Plasma Release schedule

## Beta release on the Ethereum Mainnet - ETH only era

This is a beta release to demonstrate the viability of Plasma, allow developers to start building on top of it, build a ground for liquidity market, automated software and verifying client.

**ETA:** ~15 of October 2018. Included functions:
- This is an Ethereum Mainnet (!) deployment
- Main Plasma functions such as ETH transfer. Tokens will not be supported in Beta
- Transaction structure is frozen for a whole Beta
- Deposit and exit procedure
- Initial effort for liquidity market
- Users will have a limit of 0.2 ETH to deposit their funds
- Whole set of challenges to provide an operator's fraud proof
- "Normal" exit procedure
- We will challenge all illegitimate exits on our side on initial release and encourage users to run verifying clients for network liveness

Transaction structure in the Beta epoch limits transfers to ETH only and uses one signature per transaction, so every input should belong to the same user.

The Beta stage will have the following functions activated later (~1 month preriod):
- "Limbo exits" - exits from unavailable blocks. For now it's not fully tested and no automated procedure for challenges is setup yet on our side

Over the time of Beta the following software will be finished to develop in the following priority:
- Vefirying client in two options:
    - full (like a full Ethereum node), monitors the whole UTXO space
    - simple monitors the user's associated UTXOs and ~30% of the address space
- An example of the automated liquidity provision software
- Public proof suppliers - supply Merkle proofs for transactions in block

For this release and while full verifying software is not stable and available all invalid exits will be challenged by us. With graduate improvement of network liveness we will introduce a delay to allow other users to challenge invalid exit requests.

If no errors are found testing era will last for 3 month without migration or stopping the smart-contracts functionality. After the end of the testing this implementation will either be stopped or re-released without restrictions for pure ETH transfers functionality.

## ERC20, swaps and DEX era

To extend the functionality the following changes will be applied:
- New transaction structure with:
    - separate signatures per input
    - introduction of assetType and assetAmount/ID field for token (ERC20 and ERC721) support 
    - Introduce transaction lifetime - maximum block number when it is allowed to be included in a block
    - Add new type of transaction - swap. It allows users to swap their assets
- New piece of software - marketplaces and bots
    - With a "swap" transaction type and limited transaction lifetime two parties can safely agree on exchange of their assets. 
    - Plasma operator can NOT sign a matching transaction due to problems with correctness of Plasma or an insane amount of monitoring required to cover all potential fraud proofs
    - So, it requires some discovery mechanism for two users to find each other and make a deal
    - Such mechanism will be released in a form of open marketplace software where user desired trades are posted and another party can a deal of his choice
    - To make it efficient bots will be required - user can make an order to his bot to "sell some asset at the price of X or higher" and bot will try to make a deal for him with other such bots.
- Global Plasma structure improvements:
    - "Checkpointing" - upon release of every block an operator also includes a commitment to the Sparse Merkle Tree (SMT) of either UTXO information (assetID, value and owner) or spending records (was spent in some block). There is 2 week period for a separate set of challenges that prove invalidity of such tree, and if such tree is finalized users can sync their nodes much more quickly by downloading this tree only and only 2 weeks of new blocks.
    - Mass exits feature - this is a tentative feature. Mass exit aggregation UX is not good at the moment, so this feature may be included or not.

**ETA:** Release of this software will be staged too.
- Beta ~ Q1 2018
- Release Q2 2018
