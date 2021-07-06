# Futures

Stocks, commodities and real-world tokenization will be offered on DeFiChain via futures contract, offered by [Operators](/operator).

Future is offered by operators with the assistance of price oracles, allowing prices of assets to be tracked, real world or otherwise.

DeFiChain will support 2 types of futures:

1. Fixed expiry
  - With a fixed expiry at a certain date in the future.
  - At the time of expiry,

2. Perpetual swap
  - With funding mechanics

## Mechanics

1. Margin contract
  - Trader is able to fund their Future trading position through this margin contract.
  - Contract can be funded with DFI and other tokens that operators decide, e.g. BTC.
  - Operator can also define _initial margin_ and _maintenance margin_ for a margin contract.

2. Asset
  - Operator decide assets to list.
  - Assets can be stocks, commodities or anything that has a price. It is operator's responsibility to ensure that there are good price oracles for traded assets.

3. Order
  - User is able to make an order. It can be a **long** or **short** order.
  - Orders are matched automatically on-chain.

4. Settlement
  - Fixed expiry future will be settled automatically based on settlement oracle price published.
  - Perpetual swap future has no settlement. There is instead a **funding rate mechanism**.

5. Perpetual swap funding
  - Funding rate is automatically determined by DeFiChain based on fixed algorithm: oracle price, perpetual swap trading price.
  - Funding period can be set by operator, but it should be between 1x to a max of 12x per day. For reference, most centralized perpetual swap offerings have a 8h funding interval (3x per day).

6. Liquidation
  - When net margin (total margin + unrealized PnL) is less than maintenance margin, forced liquidation would occur.
  - TBD: Either automated, operator-run, or community-run with incentives.


### Enahncements

1. Future can further be enhanced to allow leverage. DeFiChain will start with no leverage (1x) and will introduce leverage once the mechanisms are stable.

## RPC

_TK_
