# Description

This file contains a block structure. It is much more trivial than Transaction. Block header contains the following fields:
- Version, 1 byte - indicates a version of the block, so transactions of the same version are allowed inside. At some point Plasma contract can be upgraded to support higher version numbers (with backward compatibility for exits from the old versions) to add new functionality.
- BlockNumber, 4 bytes
- PreviousHeaderHash, 32 bytes - hash of the full header of the previous block
- Merkle root, 32 bytes - merkle root of the transactions tree. Transactions tree uses padding to have transaction number being deduced purely from the position in a tree. Padding is an empty transaction with no inputs, outputs and signatures.
- Signature, 65 bytes - a byte concatenation of `SignatureV, SignatureR, SignatureS` signature from the Plasma operator. Signature structure can in princible be of any kind and here is given only as an example

Header hash (for `PreviousHeaderHash` field) is calculated from byte concatenation of all fields above. If header length is constant that block is formed as 
```
    Header|RLPEncode([Transaction0, Transaction1, ...])
```

Header of non-constant length is hardly necessary and will only be added if strictly required.