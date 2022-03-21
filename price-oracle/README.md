# Price oracle

Even though Oracle could technically provide any off-chain data, the initial version of Oracle focuses on numeric data especially price feeds to augment various features and functionalities on DeFiChain, especially future, option & loan.

## Economics & game theory

To provide a fully robust trustless oracle service requires components such as reputation, penalty, and various other mechanics, it is not DeFiChain's current focus to build a full oracle system, at least at this stage. DeFiChain utilizes Operator and free economy to ensure that oracles are run as fairly as possible. It is Operator's interest to ensure that oracles that they appoint are fairly run, and users are free to select operators that they like.

## Mechanics

While truthfulness of the data cannot be ascertained on the blockchain alone, the goals of the design should focus on building a _robust_ oracle system that should continue to function fine with a few minority oracle outages.

### Price feeds

Price feed should be 8 decimal places.

Requires foundation authorization.
1. `createpricefeed BTC_USD_PRICE`
2. `removepricefeed BTC_USD_PRICE`

### Oracle appointments

Requires Operator authorization. Before Operator model is ready, it uses only 1 default [opspace](../operator) and requires foundation authorization.

1. `appointoracle ADDRESS '["BTC_USD_PRICE", "ETH_USD_PRICE"]' WEIGHTAGE`
    - returns `oracleid`. `oracleid` should be a deterministic hash and not incremental count.
    - `WEIGHTAGE` is any number between 0 to 100. Default to 20.
2. `removeoracle ORACLEID`
3. `updateoracle ORACLEID ADDRESS '["BTC_USD_PRICE", "ETH_USD_PRICE"]' WEIGHTAGE`


### Price feeds operations

Requires appointed Oracle authorization.

1. `setrawprice TIMESTAMP '{"BTC_USD_PRICE": 38293.12}'`
    - Raw price update should only be valid if timestamp is within one hour from the latest blocktime.
    - timestamp is there so that updates that are stuck in mempool for awhile does not get accepted after timeout.

### General usage

General usage, requires no authorizations.

1. `listlatestrawprices BTC_USD_PRICE`
    - Also supports `listlatestrawprices` to list across all feeds.
    - Returns an array of all the latest price updates, e.g.

    ```json
    [
        {
            "pricefeed": "BTC_USD_PRICE",
            "oracleid": "ORACLEID",
            "weightage": 50,
            "timestamp": "timestamp of the last update",
            "rawprice": 38293.12000000,
            "state": "live"
        }
    ]
    ```

    - Price updates that are older than 1 hour from the latest blocktime would have its state to be `expired`.
    - It is also imperative that the node is not storing all raw price data on database, it only needs to know the latest raw price for each oracles.

1. `getprice BTC_USD_PRICE`
    - Returns number with aggregated price.
    - Aggregate price = Sum of (latest rawprice of live oracles * weightage) / Sum (live oracles' weightage)
    - If no live oracle is found for the price, it should throw an error. It MUST NOT return 0 or large number.

1. `listprices`
    - Same as above, but an array of all feeds.
