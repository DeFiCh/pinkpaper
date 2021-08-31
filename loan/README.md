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

For example, if a vault holds $200 worth of collateral asset, with $90 minted tokens and $10 of total interest accrued, the collateralization ratio is 200 / (90 + 10) = 2.

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

- `LOAN_LIQUIDATION_PENALTY`: 0.05 (default), for 5% liquidation penalty. This is converted into DFI upon liquidation and burned.

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

Interest rate for loan consists of 2 parts:

- Vault interest, based on loan scheme of individual vaults
- Token interest, based on loan tokens, e.g. `TSLA` token might have its own interest that is chargeable only for `TSLA` token.

[DeFi fees](../fees) are burned. Fees are typically collected in the form of the loan token repayment, and automatically swapped on DEX for DFI to be burned.

For example, if a loan of 100 `TSLA` is taken out and repaid back exactly 6 months later with the APR of 2%, the user will have to repay 101 `TSLA` during redemption to regain full access of collateral in the vault.

100 `TSLA` will be burned as part of the repayment process. 1 `TSLA` that forms the interest payment will be swapped on DEX for DFI. The resulting DFI will burned. All of these occur atomically as part of DeFiChain consensus.

### Vault

User is able to freely open a vault and deposit tokens to a vault. Vault is transferable to other owners, including being controlled by multisig address.

### Collateral DFI requirement

Vault's collateral requires at least 50% of it to be DFI.

When depositing non-DFI collateral to a vault, it needs to ensure that the resulting DFI proportion of for vault's collateral is at least 50% or more, or the deposit transaction should fail.

This requirement is only checked upon deposited and does not play a role in liquidation. For instance, if at the time of posit, a vault's DFI collateral is 52%, but falls to 49% without any further deposit. This WILL NOT trigger liquidation as long as vault's minimum collateralization ratio condition is still met.

## Liquidation

Liquidation occurs when the collateralization ratio of a vault falls below its minimum. Liquidation is crucial in ensuring that all loan tokens are sufficiently over-collateralized to support its price.

For Turing-complete VM-based blockchain, liquidation requires the assistance of publicly run bots to trigger it, in exchange for some incentives.

There are two problems with that:
- During major price movements, the network fee, or commonly known as gas price, would sky rocket due to a race by both the vault owners frantically trying to save their vaults from getting liquidated, and by the public liquidators trying to trigger liquidation.
- If liquidations are not triggered on time, tokens that are generated as a result of loan runs the risk of losing its values. Worse scenario would see it getting into a negative feedback loop situation creating a rapid crash of token values.

DeFiChain, a blockchain that's built for specifically for DeFi, enjoy the benefit of automation and can have liquidation trigger automatically on-chain. This address the above two problems.

### Methodology

On every block, the node would evaluate all vaults and trigger the following automatically for liquidation.

When collateralization ratio of a vault falls below minimum collateralization ratio, liquidation would be triggered. During liquidation collateral auction is initiated.

When a vault is in liquidation mode, it can no longer be used until all the loan amount is repaid before it can be reopened. Interest accrual is also stopped during liquidation process.

#### Collateral auction

The entirety of loan and collateral of a vault if put up for auction. As the first version of collateral auction requires bidding of the full amount, auction is automatically split into batches of _around_ $10,000 worth each to facilitate for easier bidding.

#### Example

A vault, that requires a minimum of 150% collateralization ratio contains $15,000 worth of collateral, which consists of $10,000 worth if BTC and $5,000 worth if DFI, and the total loan, inclusive of interest, is $11,000 worth.

- Collateral: $10,000 BTC + $5,000 DFI
- Loan: $10,000 `TSLA` + $1,000 interest (`TSLA`)

Collateralization ratio = 15k / 11k = 136.36% less than the required 150%. Liquidation is triggered automatically by consensus. Auction will be initiated with the intention to liquidate $15k of collateral to recover $11k of loan.

Liquidation penalty is defined as `LOAN_LIQUIDATION_PENALTY`. The amount to be recovered should also include the penalty. If `LOAN_LIQUIDATION_PENALTY` is 0.05, the total amount to be recovered is thus $11k * 1.05 = $11,550

Total collateral worth is $15k, it will thus be split into 2 batches, each not exceeding $10k:

1. 2/3 of the whole vault, worth $10k of collateral.
2. 1/3 of the whole vault, worth $5k of collateral.

Specifically the following:

1. ($6,667 BTC and $3,333 DFI) for $7,700 `TSLA`
1. ($3,333 BTC and $1,667 DFI) for $3,850 `TSLA`

`TSLA` recovered at the conclusion of the auction will be paid back to the vault. Once all loan is repaid, vault will exit liquidation state, allowing its owner to continue to use it. It will, however, not terminate all opened auctions. Opened auctions see through to their conclusion.


## Collateral auction

Collateral auction will run for 720 blocks (approximately 6 hours) with the starting price based on liquidation vault's ratio.

Using the same illustration, Auction 1: ($6,667 BTC and $3,333 DFI) for $7,700 `TSLA`

Anyone can participate in the auction, including the vault owner by bidding for the entirety of the auction with higher amount. To prevent unnecessary auction sniping at closing blocks, each higher bid, except the first one, has to be at least 1% higher than previous bid.

A user could bid for the same auction by offering $7,800 worth of `TSLA` tokens. During bidding, the bid is locked and refunded immediately when outbid. Top bid is not cancelable.

At the conclusion of the auction, the entirety of the for sale collateral will be transferred to the winner – in this case the winner would be paying $7,800 for $10,000 worth of BTC and DFI.

$7,800 worth of `TSLA` token will be processed as follow:
- Paid back to vault = $7,800 worth of `TSLA` / (1 + `LOAN_LIQUIDATION_PENALTY`) = 7800 / 1.05 = $7,428.57 `TSLA` token
- The remaining, i.e. liquidation penalty, will be swapped to DFI and burned. $371.43 worth of `TSLA` be swapped as follows and  trackably burned:
    1. `TSLA` to `USD` from `TSLA-USD` DEX.
    1. `USD` to `DFI` from `USD-DFI` DEX.

If auctions yield more `TSLA` than the vault's loan, it will be deposited back to the vault as part of vault's asset, but not collateral. It can be withdrawn from the vault after vault exits liquidation state by vault's owner and carries no interest.

### Multiple loan tokens

If a vault consists of multiple loan tokens, they should be auctioned separately.

For instance, a liquidating vault that consists of the following:
- Collateral: 1 BTC & 100 DFI
- Loan: $10k worth of USD * $40k worth TSLA

would be liquidated as follows:

- (0.2 BTC & 20 DFI) for $10k worth of USD
- (0.8 BTC & 80 DFI) for $40k worth of TSLA

and further split into $10k batches.


### Failed auction

Auction that expires after 720 blocks without bids will be reopened immediately based on the remaining amount of collateral and loan that the vault receives due to conclusion of other auction batches.


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

1. `setloantoken DATA`
    - Creates or updates loan token.
    - `DATA` (JSON)
        - `symbol`
        - `name`
        - `priceId`: ID of to tie the loan token's price to.
        - `mintable` (bool): When this is `true`, vault owner can mint this token.
        - `interestrate`: Annual rate, but chargeable per block (scaled to 30-sec block). e.g. 0.035 for 3.5% interest rate. Must be >= 0. Default: 0.
    - To also implement `updateloantoken` and `listloantokens`.

### Public

Requires no authorizations.

1. `getloaninfo`
    - Returns the attributes of loans offered by Operator.
    - Includes especially the following:
        - Collateral tokens: List of collateral tokens
        - Loan tokens: List of loan tokens
        - Loan schemes: List of loan schemes
        - Collateral value (USD): Total collateral in vaults
        - Loan value (USD): Total loan token in vaults

1. `listloanschemes`
    - Returns the list of attributes to all the available loan schemes.

#### Vault

Vault-related, but does not require owner's authentication.

1. `createvault [OWNER_ADDRESS] [SCHEME_ID]`
    - Create a vault for `OWNER_ADDRESS`.
    - No `OWNER_ADDRESS` authorization required so that it can be used for yet-to-be-revealed script hash address.
    - If `OWNER_ADDRESS` is not specified in RPC, generates a new address from wallet before crafting the transaction.

1. `deposittovault VAULT_ID TOKEN_TO_DEPOSIT`
    - Deposit accepted collateral tokens to vault.
    - Does not require authentication, anyone can top up anyone's vault.
    - `TOKEN_TO_DEPOSIT` should be in similar format as other tokens on DeFiChain, e.g. `23.42@USDC` and `0.42@DFI`.
    - Also take note on the >= 50% DFI requirement, when depositing a non-DFI token, it should reject if the total value of the vault's collateral after the deposit brings DFI value to less than 50% of vault's collateral.
    - This also means that the first deposit to a vault has to always be DFI.

1. `loanpayback VAULT_ID`
    - Pay back of loan token.
    - Only works when there are loan in the token.
    - Refunds additional loan token back to original caller, if 51.1 `TSLA` is owed and user pays 52 `TSLA`, 0.9 `TSLA` is returned as change to transaction originator (not vault owner).
    - Does not require authentication, anyone can payback anyone's loan.

1. `getvault [VAULT_ID]`
    - Returns the attributes of a vault.
    - Includes especially the following:
        - Owner address
        - Loan scheme ID
        - `isliquidated` (boolean) whether if a vault has been liquidated.

1. `listvaults`
    - Returns the list of attributes to all the created vaults.

### Vault owner

Requires ownerAddress authentication, and vault MUST NOT be in liquidation state.

1. `updatevault VAULT_ID DATA`
    - `DATA` (JSON) may consists of the following:
        - `scheme`: Allows vault owner to switch to a different scheme
        - `ownerAddress`: Transfers vault's ownership to a new address.

1. `withdrawfromvault VAULT_ID`
    - Withdraw collateral tokens from vault.
    - Vault collateralization ratio must not be less than `mincolratio` of vault's scheme.
    - Also used to withdraw assets that are left in a vault due to higher auction yield. See [Collateral auction](#collateral-auction)

1. `takeloan VAULT_ID DATA`
    - `DATA` would spell out the tokens wanting to be taken out as loan.
    - Vault collateralization ratio must not be less than `mincolratio` of vault's scheme.

### Auction

Requires no additional authentications.

1. `listauctions`

1. `bid AUCTION_ID TOTAL_PRICE`
    - `TOTAL_PRICE` of token in auction will be locked up until the conclusion of the auction, or refunded when outbid.
