# Operator

## Generalization

Generalization effort in DeFiChain aims at separation between the role of blockchain and operator, the party in charge of offering financial products.

The generalization goals are to allow anyone to create and run as operator on DeFiChain, without requiring any prior permissions from DeFiChain Foundation or its developers.

### Opspace

Operator operates exclusively within their Opspace and does not interfere with the operations of other Opspaces and entire blockchain ecosystem.

To ensure operation continuity, the current asset tokens, and any other offerings that are available prior to the introduction of Operator, will be migrated to the first Opspace when the functionality is available.

### Role

The role and functionalities available to operators include:

1. Price oracle
  - Setting of price feeds.
  - Appointing of oracles.
2. Asset token
  - Determining and setting of asset tokens that are available within an opspace.
  - Previously referred to as "DeFi Asset Token" on white paper.
3. Decentralized loan
  - Determining loan attributes: collateral types, interest rates, etc
4. Futures
  - Determining future offerings & attributes

... and others as they are made available.

Operator is also free to determine their own Operator fee for all the products that they provide.

Note that operators may only want to participate in a subset of the operations available, for example an operator may only be interested in offering decentralized loan but not interested in offering futures.


## Operations

RPC is designed work with multisignatory opcodes of Bitcoin's script, allowing operator to have clear governance over its policies.

1. `createoperator`
    - Costs 1000 DFI in DeFi fee, burned.

2. `updateoperator`
