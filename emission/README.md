# DFI emission rate

## Current

From [white-paper](https://defichain.com/white-paper/#dfi-coin):

> DeFiChain is initially launched with a 200 DFI block reward, of which 10% goes to the community fund. The Foundation pledges to guarantee this 200 DFI block reward for at least 1,050,000 blocks since the the first genesis block, so approximately 1 year.

The white paper lays out the expectation for the first year that the block reward is flat at 200 DFI. 

2 DFIPs have since been made and implemented that affected the allocation of the 200 DFI, while the total emission remains consistent at 200 DFI.

- [DFIP #1: Bitcoin anchor reward source and mechanics adjustments](https://github.com/DeFiCh/dfips/issues/1) where 0.1 DFI is reduced from community fund to cater for anchor reward
- [DFIP #2: DeFi incentive funding](https://github.com/DeFiCh/dfips/issues/2) where the original 180 DFI per block mining reward is reduced to 135 DFI 

To keep the total supply of DFI capped at 1.2 billion, the emission rate of DFI cannot stay consistent forever and will have to be reduced over time.

## Foundation and Community Fund

To further decentralize the governance of DeFiChain, the Foundation's role must be diminished significantly, and should not be holding any DFI. As such, it is proposed for the Foundation tokens to be burned entirely so no individuals or entities hold the private keys to the Foundation's DFI.

Community Fund today accumulates at 19.9 DFI per block and requires community funding proposal (CFP) to be made and votes to be called for the fund to be _manually_ released. 

Community Fund should be removed from address with private key to consensus-managed on-chain voting.

## Emission rate

### Goals

1. Maintain the same emission rate on the first cycle.
2. Avoid the dramatic "halving" like Bitcoin's.
3. Maintain the 1.2 billion DFI cap.
4. Maintain a steady expected emission rate for foreseeable future.

### Current rate

The current emission rate of DFI is 258.1 DFI per block – some of it are funded by airdrop fund.

| Description | DFI per block |
| ---- | ------ |
| Mining reward | 135 |
| Community fund | 19.9 |
| Anchor reward | 0.1 |
| _DeFi incentives_ | |
| BTC-DFI | 80 |
| ETH-DFI | 15 |
| USDT-DFI | 5 |
| LTC-DFI | 2 |
| BCH-DFI | 1 |
| DOGE-DFI | 0.1 |
| **TOTAL** | 258.1 |

The same rate will be retained on the first cycle. 

On top of the above, the following will also be pre-allocated on the first cycle:

| Description | In use today? | DFI per block | Proportion |
| ---- | ---- | ------ | ---- |
| Mining reward | Yes | 135 | 33.33% (guaranteed) |
| Community fund | Yes | 19.9 | 4.91% |
| Anchor reward | Yes | 0.1 | 0.02% | 
| DEX liquidity mining | Yes | 103.1 | 25.45% |
| Atomic swap incentives | No | 50 | 12.34% |
| Futures incentives | No | 50 | 12.34% |
| Options incentives | No | 40 | 9.88% |
| Unallocated incentives | No | 6.94 | 1.71% |
| **TOTAL** |  | 405.04 | 100% |

### Cycle & Reduction

Emission rate will start at 405.04 DFI on the first cycle so that it flows smoothly with no change before the implementation. The increment on the first cycle is coming from the [destruction of Foundation-owned DFI](https://github.com/DeFiCh/dfips/issues/17).

Cycle period is 32,690 blocks, at 37 seconds per block, each cycle takes approximately 2 weeks. 

Every cycle, the block reward reduces by 1.658%. The splitting of the rewards for respective incentives will be maintained at the same proportion as laid out on the schedule above.

Unused reward will be burned. Burned rewards will not be re-introduced, therefore counted towards the 1.2 billion supply cap.

Rewards proposed may be adjusted with future on-chain governance changes, however, masternode (mining) rewards are guaranteed to be 33.33% of the total block reward.



