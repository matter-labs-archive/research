# Description

This file contains a transaciton structure. Plasma construction itself is Move Viable Plasma with a proper exit game to ensure correctnes. It follows a UTXO model and have two different types of transactions:
- Deposit
- Transfer

Allowed number of inputs and outputs, as well as transaction structure is described below.

This implementation will have a capability to operate on different kinds of assets (ERC20, ERC721) and perform transactions that touch assets of different kinds.

## Transaction Input

This structure described an asset that is spent by the transaction. Each input spending is signed separately, that allows to manipulate on assets of different kinds.

Each input has the following fields:
- BlockNumber, 4 bytes - refers to the block number where a corresponding UTXO (that is spent by this transaction as this input) was produced
- TransactionNumber, 4 bytes - refers to the transaction number in block where a corresponding UTXO was produced
- OutputNumber number, 4 bytes - refers to the output number in transaction where a corresponding UTXO was produced
- AssetID, 32 bytes - enumerates an assetID of this UTXO. While it could have been possible to get this value by looking up a combination `BlockNumber|TransactionNumber|OutputNumber`, it allows for easier processing and partially requires a sender to have data available about the UTXO details. In Ethereum network the assetID is made as 12 zero bytes concatenated with 20 bytes of address of the corresponding ERC20 or ERC721 contract. Address equal to zero is treated as Ethereum itself. It's possible and reasonable to connect Plasma to more than one blockchain, so top 12 bytes will be used to indicate the chainID in some form
- AssetValue, 32 bytes - indicates either an amount of ERC20 tokens (or ETH) being spent or unique identifier of ERC721 token

To produce the total input first the input data is produced
```
inputData = RLPEncode([BlockNumber, TransactionNumber, OutputNumber, AssetID, AssetValue])
```

## Transaction Output

This structure described an output that is produced by transaction. 

Each output has the following fields:
- Owner, 20 bytes - indicates an owner of a produced UTXO. Due to the wide adoption of `secp256k1` and blockchains based on Ethereum it's a default choice and address is generated as `keccak256(pubkey)[12..<32]`
- AssetID, 32 bytes - enumerates an assetID of this UTXO. Rules are the same as for an input
- AssetValue, 32 bytes - indicates either an amount of ERC20 tokens (or ETH) or unique identifier of ERC721 token

To produce the total output data one should do
```
outputData = RLPEncode([Owner, AssetID, AssetValue])
```

## Transaction

First one should describe transaction types:
- Deposit - used by the Plasma operator to create a new UTXO for a user that has deposited his funds via the smart-contract. This transaction type has a numeric encoding of `1<<0`. Rules for such transaction:
    - One input signed by the Plasma operator. `BlockNumber = 0, TransactionNumber = 0, OutputNumber = 0`. `AssetID` has it's lowest 20 bytes zeroed with top 12 indicating the chainID. `AssetValue` corresponds to the unique depositID that is stored by the corresponding smart-contract.
    - One output that produces an asset with a correct `Owner`, `AssetID` and `AssetValue`

- Transfer - used by **two** parties to **exchange** their assets. Inputs to this transaction can be signed by different private keys. This transaction should have up to **four** inputs and **five** outputs to have a size that is possible to work with on-chain. Last input has special meaning and can be only used for a fee payment. This transaction type has a numeric encoding of `1<<1`. Last output is intended to be sent to the Plasma operator for fee payment.

To allow graduate upgradeability of Plasma and prevent "second opinion" for counterparties in "exchange" transaction and an operator himself the transaction structure is the following:
- Version, 1 byte - indicates a version of the transaction. For transaction version `1` all allowed types are listed above
- Type, 1 byte - encodes a transaction type
- GoodUntil, 4 bytes - indicates a block number up to (an including) which transaction can be included. It prevents a Plasma operator from censoring a transaction and griefing on exit by finally including it into the block, and also prevents parties in a `Swap` transaction from risk of never executing a deal
- Inputs, RLP encoded list of encoded Transaction Inputs
- Outputs, RLP encoded list of encoded Transaction Outputs
- MessageData, either empty or 32 bytes - allows to attach some message (or hash of a long message) to the transaction. Is never a part of exit process

This is a main body of the transaction. For signature purposes it's encoded as 
```
transactionData = RLPEncode([Version, Type, GoodUntil, Inputs, Outputs, MessageData])
```

Then signatures are required from every participant whose input was a part of the transaction.

- Signatures, a list of lists `[signatureV, signatureR, signatureS]` - inducated a will of input owners to do a transaction. A `transactionData` is signed separately by each owner and than signatures are concatenated. If one owner brought more than one input an extra signature is not required.

Fee payments for transaction depend on a "branching factor" - a difference between consumed inputs and produced outputs. For transactions with a branching factor of less than zero fee is not taken.