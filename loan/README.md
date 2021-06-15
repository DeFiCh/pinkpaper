# Loan and Decentralized Tokenization

Loan is be offered by [operator](../operator) to allow decentralized price-tracking tokens to be created by users.

Before Operator model is ready, it uses only 1 default [opspace](../operator) and requires foundation authorization, that acts on behalf of community.

Possibility of tokens being offered decentrally can be of many types, including but not limited to:

1. Cryptocurrency such as `BTC`, `ETH`, `LTC`, etc.
1. Stablecoin such as `USD`, `EUR` tokens, etc.
1. Stocks token such as `TSLA`, `MSFT`, etc.
1. Other asset tokens.

## Concepts

## Collateralization

Loan on DeFiChain operates on over-collateralization mechanism. User first has to open a vault.

### Operator

Operator plays to role to decide on the following:

1. Collateral tokens and their respective collateralization values.
  - For instance, an Operator can decide to allow users to start a Vault in their opspace with the following collateralization factors:
    - `DFI` at 100% collateralization factor
    - `BTC` at 100% collateralization factor
    - `DOGE` at 50% collateralization factor

Operator, also via opspace's governance values, can decide the following:

- `LOAN_MIN_COLLATERIZATION_RATIO`: 1.5 (default), for 150% collateralization ratio. _Deprecated for loan scheme attribute_
- `LOAN_LIQUIDATION_PENALTY`: 0.15 (default), for 15% liquidation penalty.

### Scheme

Loan scheme allows an opreator to set up different loan schemes under the same Operator. Loan schemes allows an operator to define the following:

  - Collateral token and different collateralization factor
  - Loan interest rates

This will be introduced alongside Operator.

### Vault

User is able to freely open a vault and deposit tokens to a vault. Vault is transferable to other owners, including being controlled by multisig address.


## RPC

### Operator

Requires Operator authorization. Before Operator model is ready, it uses only 1 default [opspace](../operator) and requires foundation authorization.

#### Loan scheme

1. `createloanscheme DATA`
  - `DATA` (JSON) can consist of the following:
    - `mincolratio`: e.g. 1.75 for 175% minimum collateralization ratio. Cannot be less than 1.
    - `interestrate`: Annual rate, but chargeable per block (scaled to 30-sec block). e.g. 0.035 for 3.5% interest rate. Must be >0.
    - `id`: Non-colliding scheme ID that's unique within opspace, e.g. `MIN_175`

1. `updateloanscheme SCHEME_ID DATA [ACTIVATE_AFTER_BLOCK]`
  - Update loan scheme details.
  - `ACTIVATE_AFTER_BLOCK` _(optional)_: If set, this will only be activated after the set block. The purpose is to allow good operators to provide sufficient warning.

1. `destroyloanscheme SCHEME_ID [ACTIVATE_AFTER_BLOCK]`
  - Destroy a loan scheme.
  - `ACTIVATE_AFTER_BLOCK` _(optional)_: If set, this will only be activated after the set block. The purpose is to allow good operators to provide sufficient warning.
  - Any loans under the scheme will be moved to default loan scheme after destruction.

1. `setdefaultloanscheme SCHEME_ID`
  - Designate a default loan scheme from the available loan schemes.

#### General

1. `setcollateraltoken TOKEN FACTOR PRICE_FEED_ID [ACTIVATE_AFTER_BLOCK]`
  - `TOKEN`: Token must not be the same decentralized token is issued by the same operator's loan program.
  - `FACTOR`: A number between `0` to `1`, inclusive. `1` being 100% collaterization factor.
  - `PRICE_FEED_ID`: Price feed from [oracle](../oracle) that is being used to price the token.
  - `ACTIVATE_AFTER_BLOCK` _(optional)_: If set, this will only be activated after the set block. The purpose is to allow good operators to provide sufficient warning to their users should certain collaterization factors need to be updated.
  - This very same transaction type is also used for updating and removing of collateral token, by setting the factor to `0`.

1. `setgentoken DATA`
  - Creates or updates generator token – token generated from loan.
  - `DATA` (JSON)
    - `symbol`
    - `name`
    - `priceId`: ID of to tie the generator token's price to.
    - `mintable` (bool): When this is `true`, vault owner can mint this token.
  - This very same transaction type is also used for updating and removing of generator token, by setting the factor to `0`.

### Public

Requires no autohrizations.

1. `getloan [OPERATOR_ID]`
  - Returns the attributes of loan offered by Operator.

1. `listloanschemes`
  - Returns the list of attributes to all the available loan schemes.


#### Vault

Vault-related, but does not require owner's authentication.

1. `createvault ownerAddress [SCHEME_ID]`
  - Create a vault for ownerAddress.
  - No ownerAddress authrotization required so that it can be used for yet-to-be-revealed script hash address.

1. `depositToVault`
  - Deposit accepted collateral tokens to vault.
  - Does not require authentication, anyone can top up anyone's vault.

### Vault owner

Requires ownerAddress authentication.

1. `updateVault VAULT_ID DATA`
  - `DATA` (JSON) may consists of the following:
    - `scheme`: Allows vault owner to switch to a different scheme
    - `ownerAddress`: Transfers vault's ownership to a new address.

1. `withdrawFromVault VAULT_ID`
  - Withdraw collateral tokens from vault.
  - Vault must not by less than `mincolratio` of vault's scheme.

1.
