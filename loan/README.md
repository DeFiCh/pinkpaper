# Loan and Decentralized Tokenization

Loan is be offered by [operator](../operator) to allow decentralized price-tracking tokens to be created by users.

Before Operator model is ready, it uses only 1 default [opspace](../operator) and requires foundation authorization, that acts on behalf of community.

Possibility of tokens being offered in a decentralized manner can be of many types, including but not limited to:

1. Cryptocurrency such as `BTC`, `ETH`, `LTC`, etc.
1. Stablecoin such as `USD`, `EUR` tokens, etc.
1. Stocks token such as `TSLA`, `MSFT`, etc.
1. Other asset tokens.

## Concepts

### Collateralization ratio

Loan on DeFiChain operates on over-collateralization mechanism. User first has to open a vault to mint tokens.

Collateralization is calculated for each vaults, using the following formula:

```
Collateralization ratio =
    Total effective value of collateral /
    (Total value of tokens minted + Total interest)
```

For example, if a vault holds $200 worth of collateral asset, with $130 minted tokens and $10 of total interest accrued, the collateralization ratio is 200 / (90 + 10) = 2.

This is not to be confused with _collateralization factor_.

### Collateralization factor

Assets that are accepted for collateralization could be accepted at different collateralization factor, from 0% to 100%. When an asset collateralization factor is 100%, the entirety of the asset's value contributes to the collateralization value of the vault, however, if an asset collateralization factor is say 50%, then only half of its value contributes to the total vault's collateral.

For example if `DOGE` is accepted at 70% collateralization factor, $100 of `DOGE` would contribute to $70 of collateral value in a vault.

This is not to be confused with _collateralization ratio_.

### Operator

Operator plays to role to decide on the following:

1. Collateral tokens and their respective collateralization values.
    - For instance, an Operator can decide to allow users to start a Vault in their opspace with the following collateralization factors:
        - `DFI` at 100% collateralization factor
        - `BTC` at 100% collateralization factor
        - `DOGE` at 70% collateralization factor

Operator, also via opspace's governance values, can decide the following:

- `LOAN_LIQUIDATION_MARGIN`: 0.1 (default), for 10% liquidation margin. If set at 10%, during liquidation event, additional 10% are liquidated so there's around 10% margin above minimum collateralization ratio.
- `LOAN_LIQUIDATION_PENALTY`: 0.1 (default), for 10% liquidation penalty. This is converted into DFI upon liquidation and burned.

### Loan scheme

Loan scheme allows an operator to set up different loan schemes under the same Operator. Loan schemes allows an operator to define the following:

- Collateral token and different collateralization factor
- Loan interest rates

For example, a series of loan schemes could be made available:

1. Min collateralization ratio: 150%. Interest rate: 5% APR.
1. Min collateralization ratio: 175%. Interest rate: 3% APR.
1. Min collateralization ratio: 200%. Interest rate: 2% APR.
1. Min collateralization ratio: 350%. Interest rate: 1.5% APR.
1. Min collateralization ratio: 500%. Interest rate: 1% APR.
1. Min collateralization ratio: 1000%. Interest rate: 0.5% APR.

Vault owner can define which loan scheme to subscribe to during initial vault creation and can move it freely after that. Take caution though that if you move to a scheme with lower collateralization ratio, liquidation might be triggered.

### Interest rate

Interest rate for loan is chargeable in two forms:

- DeFi fee, which is required and it must be in DFI
- Operator fee, which can be in the form of any tokens

During the initial introduction, Operator fee will not be supported yet, until Operator is in place.

[DeFi fees](../fees) are burned. Fees are typically collected in the form of the loan token repayment, and automatically swapped on DEX for DFI to be burned.

For example, if a loan of 100 TSLA is taken out and repaid back exactly 6 months later with the APR of 2%, the user will have to repay 101 TSLA during redemption to regain full access of collateral in the vault.

100 TSLA will be burned as part of the repayment process. 1 TSLA that forms the interest payment will be swapped on DEX for DFI. The resulting DFI will burned. All of these occur atomically as part of DeFiChain consensus.


### Vault

User is able to freely open a vault and deposit tokens to a vault. Vault is transferable to other owners, including being controlled by multisig address.

## Liquidation

Liquidation occurs when the collateralization ratio of a vault falls below its minimum. Liquidation is required to take place in order to bring up the collateralization ratio of a vault again so the token price remains consistent.

For Turing-complete VM-based blockchain, liquidation requires the assistance of publicly run bots to trigger it, in exchange for some incentives.

There are two problems with that:
- During major price movements, the network fee, or commonly known as gas price, would sky rocket due to a race by both the vault owners frantically trying to save their vaults from getting liquidated, and by the public liquidators trying to trigger liquidation.
- If liquidations are not triggered on time, tokens that are generated as a result of loan runs the risk of losing its values. Worse scenario would see it getting into a negative feedback loop situation creating a rapid crash of token values.

DeFiChain, a blockchain that's built for specifically for DeFi, enjoy the benefit of automation and can have liquidation trigger automatically on-chain. This address the above two problems.

### Methodology

On every block, the node would evaluate all vaults and trigger the following automatically for liquidation.

When collateralization ratio of a vault falls below minimum collateralization ratio, liquidation amount will be determined as follows:

```
Total loan  = (Total value of tokens minted + Total interest)

(Collateral - Base reduction) / (Total loan - Base reduction) =
    Minimum collateralization ratio.

Base reduction =
    ((Min. ratio * Total loan) - Total collateral) /
    (Min. ratio - 1)

Effective liquidation =
    Base reduction * (1 + LOAN_LIQUIDATION_MARGIN + LOAN_LIQUIDATION_PENALTY)
```

#### Example

A vault, that requires a minimum of 150% collateralization ratio contains $150 worth of collateral and the total loan, inclusive of interest, is $110 worth.

Collateralization ratio = 150 / 110 = 136.36% less than the required 150%. Liquidation is triggered automatically by consensus.

Base reduction = (1.5 * 110 - 150) / (1.5 - 1)
    = 15 / 0.5
    = $30

By simply liquidating base reduction, the total collateralization would be 150%. However, without margin, node will constantly having to liquidate every few blocks; besides, the penalty isn't yet applied.

For illustration, collateralization ratio after base reduction = (150 - 30) / (110 - 30) = 120 / 80 = 1.5

Effective liquidation is actually the following:

- Actual liquidation = base reduction * (1 + `LOAN_LIQUIDATION_MARGIN`) = $30 * 1.1 = $33
- Liquidation penalty = base reduction * `LOAN_LIQUIDATION_PENALTY` = $30 * 0.1 = $3

A total of $33 + $3 worth of collaterals will be removed from collateral of the vault.

### Collateral removal

The order of collateral selection for removal would be in the following order:

1. Non-DFI collateral with the highest liquidity in the DEX.
2. DFI collateral.

If a vault has BTC and DFI as collateral, BTC would be used for liquidation first until there are no more BTC, then DFI will be used.

From the above example, say in this case $33 + $3 worth of BTC that will be liquidated. Assuming also that the vault takes out a loan of `TSLA` token and `USD` token, with `TSLA` being the larger position.

They will be immediately trigger the following actions, all on consensus:

1. Liquidation penalty: $3 worth of `BTC` will be swapped on BTC-DFI DEX for DFI and the resulting DFI be trackably burned.
2. Vault payback: $33 worth of `BTC` will be liquidated and swapped to `TSLA` for payback. As `TSLA` DEX could be in the form of `TSLA-USD`, the DEX path would be:
    1. `BTC` to `DFI` from `BTC-DFI` DEX.
    1. `DFI` to `USD` from `USD-DFI` DEX.
    1. `USD` to `TSLA` from `TSLA-DFI` DEX.

The above swaps are atomically carried out by blockchain consensus. The resulting `TSLA` token is paid back to the vault to reduce the loan position.

### Effect

Liquidation is triggered based on oracle's price and not on DEX prices, to prevent DEX manipulation.

The effect of the automatic liquidation that swaps on DEX assists in tracking of real world asset price to DEX price, especially if token is trading below real world price.

However, if asset is trading on DEX above real world price, especially above `LOAN_LIQUIDATION_MARGIN` more than real world price, there is a risk of automatic liquidation would not yield sufficient asset token to keep the vault afloat, triggering a spiral of liquidation every block that eventually drains the entire collateral.

_TK: Switching to auction model_


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
    - `FACTOR`: A number between `0` to `1`, inclusive. `1` being 100% collateralization factor.
    - `PRICE_FEED_ID`: Price feed from [oracle](../oracle) that is being used to price the token.
    - `ACTIVATE_AFTER_BLOCK` _(optional)_: If set, this will only be activated after the set block. The purpose is to allow good operators to provide sufficient warning to their users should certain collateralization factors need to be updated.
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

Requires no authorizations.

1. `getloan [OPERATOR_ID]`
    - Returns the attributes of loan offered by Operator.

1. `listloanschemes`
    - Returns the list of attributes to all the available loan schemes.


#### Vault

Vault-related, but does not require owner's authentication.

1. `createvault ownerAddress [SCHEME_ID]`
    - Create a vault for ownerAddress.
    - No ownerAddress authorization required so that it can be used for yet-to-be-revealed script hash address.

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
