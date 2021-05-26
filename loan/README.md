# Loan and Decentralized Tokenization

Loan is be offered by [operator](../operator) to allow decentralized price-tracking tokens to be created by users.

Before Operator model is ready, it uses only 1 default [opspace](../operator) and requires foundation authorization, that acts on behalf of community.

Possibility of tokens being offered decentrally can be of many types, including but not limited to:

1. Cryptocurrency such as `BTC`, `ETH`, `LTC`, etc.
1. Stablecoin such as `USD`, `EUR` tokens, etc.
1. Stocks token such as `TSLA`, `MSFT`, etc.
1. Other asset tokens.

## Concepts

## Operator

Operator plays to role to decide on the following:

1. Collateral tokens and their respective collateralization values.
  - For instance, an Operator can decide to allow users to start a Vault in their opspace with the following collateralization factors:
    - `DFI` at 100% collateralization factor
    - `BTC` at 100% collateralization factor
    - `DOGE` at 50% collateralization factor

Operator, also via opspace's governance values, can decide the following:

- `LOAN_MIN_COLLATERIZATION_RATIO`: 1.75 (default), for 175% collateralization ratio.
- `LOAN_LIQUIDATION_PENALTY`: 0.15 (default), for 15% liquidation penalty.

## Vault

## RPC

### Operator

Requires Operator authorization. Before Operator model is ready, it uses only 1 default [opspace](../operator) and requires foundation authorization.

1. `setcollateraltoken TOKEN FACTOR PRICE_FEED_ID [ACTIVATE_AFTER_BLOCK]`
  - `TOKEN`: Token must not be the same decentralized token is issued by the same operator's loan program.
  - `FACTOR`: A number between `0` to `1`, inclusive. `1` being 100% collaterization factor.
  - `PRICE_FEED_ID`: Price feed from [oracle](../oracle) that is being used to price the token.
  - `ACTIVATE_AFTER_BLOCK` _(optional)_: If set, this will only be activated after the set block. The purpose is to allow good operators to provide sufficient warning to their users should certain collaterization factors need to be updated.
  - This very same transaction type is also used for updating and removing of collateral token, by setting the factor to `0`.
