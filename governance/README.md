# On-chain Governance

## Governance summary

Voting is carried out every 70,000 blocks, roughly once every month.

Types of governance:

1. Community development fund request proposal
    - Requesting of community development fund.
    - Requires simple majority (> 50%) by non-neutral voting masternodes to pass.
    - Fee: 10 DFI per proposal, of which half is burned.
    - The other half of the fee is distributed equally among all voting masternodes to encourage voting.

2. Block reward reallocation proposal
    - Note the miner's reward (PoS) are guaranteed and cannot be reallocated.
    - Requires super majority (> 66.67%) by non-neutral voting masternodes to pass.
    - Fee: 500 DFI, of which half is burned.
    - The other half of the fee is distributed equally among all voting masternodes to encourage voting.

3. Vote of confidence
    - General directional vote, not consensus enforcible.
    - Requires super majority (> 66.67%) by non-neutral voting masternodes to pass.
    - Fee: 50 DFI per proposal, of which half is burned
    - The other half of the fee is distributed equally among all voting masternodes to encourage voting.


## Community fund request proposal (CFP)

### Current

Currently every block creates 19.9 DFI to community fund address that is controlled by Foundation.

### Improvements

The amount at `dZcHjYhKtEM88TtZLjp314H2xZjkztXtRc` will be moved to a community balance, we call this the "Community Development Fund".

This fund, will be similar to Incentive Funding and Anchor Reward, they have no private keys and are unrealized by default unless allocated.

Voting is carried our every 70,000 blocks (at \~2335 blocks per day, that's roughly once every calendar month). This should be configurable as chainparams and requires a hardfork to update. The block where the vote is finalized is also known as "Voting Finalizing Block".

User is able to submit a fund request that is to be voted by masternodes. Each fund request costs non-refundable 10 DFI. The 10 DFI is not burned, but being added to the proposal for that will be paid out to all voting masternodes to encourage participation. Each fund request can be also specify `cycle`. Cycle is defaulted to `1` for one-off fund request, this also allows for periodic fund requests where a request is for a fixed amount to be paid out every cycle (every 70,000 blocks) for the specified cycles if the funding request remains APPROVED state.

### Data structure of a funding request

#### User submitted data
1. Title (required, this may also be implemented as hash of title to conserve on-chain storage)
1. Payout address (required) this can be different from requester's address
1. Finalize after block (optional, default=curr block + (blocks per cycle / 2))
    - Max valid value would be curr block + 3 * blocks per cycle. No minimum.
    - The rationale for this value is to allow sufficient time for requests to be discussed and voted on, esp. when it is submitted closer to the end of current voting cycle.
1. Amount per period (in DFI)
1. Period requested (required, integer, default=1)

#### Internal data
1. Completed payment periods (integer)
1. State
1. Voting data


#### States of funding request

Funding requests can have one of the following states

1. `VOTING`: Successfully submitted request, currently undergoing voting state
1. `REJECTED`: At voting finalizing block, if a request does not achieve. This is a FINAL state and cannot be re-opened.
1. `COMPLETED`: A funding request that has been paid out entirely. When completed payment periods is equals to period requested, request is marked as `COMPLETED`. . This is a FINAL state and cannot be re-opened.

The absence of the state `APPROVED` is intended. An approved request will simply have its payout made and increment the completed payment period. If payment has been completed fully, request state will be changed to `COMPLETED`, which is a final state. A corollary to this allows for a request that requested for a payout of 1000 DFI every month and has received for 2 months to be rejected at the 3rd month if there are more no votes to yes votes.



### RPC

1. `creategovcfr '{DATA}'`
    - `DATA` is serialized JSON with the following:
        - `title`
        - `finalizeAfter`: Defaulted to current block height + 70000/2
        - `cycles`: Defaulted to `1` if unspecified.
        - `amount`: in DFI.
        - `payoutAddress`: DFI address
    - 10 DFI is paid along with the submission.
    - Request is finalized after submission, no edits are possible thereafter.
    - Returns `txid`. The `txid` is also used as the proposal ID.

1. `vote PROPOSALID DECISION`
    - `PROPOSALID` is the `txid` of the submitted funding request.
    - `DECISION` can be either of the following:
        - `yes`
        - `no`
        - `neutral`
    - A masternode is able to change the vote decisions before the request is finalized, the last vote as seen on the block BEFORE finalizing block is counted.

## Votes

A masternode is deemed to be eligible to cast a vote if ALL the following conditions are satisfied:

1. The masternode is in `ENABLED` state during vote submission AND at vote finalizing block.
1. The masternode has successfully minted at least 1 block during vote submission AND at vote finalizing block.

Any eligible masternodes can cast a vote to any proposal that is in `VOTING` state, regardless of completed payment period.

Masternodes are able to cast any of the following votes:

1. YES
2. NO
3. NEUTRAL

At every vote finalizing block, a community fund proposal (CFP) is deemed to be eligible to be paid out of all the followings are true:

1. There are more YES votes than NO votes. Simple majority wins. NEUTRAL votes are not counted. If there are equal amount of YES and NO votes, the CFR is deemed to have failed and rejected.
2. The voting masternodes eligibility must be `ENABLED` at the finalizing block and have mined at least 1 block.

### Voting incentives

One of the problems of governance is that masternodes are not directly incentivized to vote as they are rewarded with staking but usually not with voting. To encourage masternodes participation in governance voting, 50% of the fees are distributed to all voting masternodes for each DFIPs. All voting masternodes, be it `yes`, `no`, or `neutral` are all eligible to receive a share of the fee, distributed evenly.


## Other proposals

Other proposals such as block reallocation proposal and vote of confidence will follow the same voting cycle.

## Supplementary website

While blockchain's as data storage is desired to maintain impartiality and anonymity of discussions, it is not a conducive medium for a wider discussion. A supplementary website will be made available to facilitate all proposal discussions and track the progress.

A good supplementary website is suggested to carry the following features:

1. **Proposer identification** to facilitate meaningful discussion among the community and the proposer. This does not, however, imply the need of a user account system.
2. **Proposal extension and title hash validation**. As the blockchain only contains the hash of title, the supplementary website must allow the proposer to enter the full proposal title hash and enter any additional descriptions on the proposals.
3. **Vote tracking**. Ability to track votes as it happens.

An example good supplementary website would be https://www.dashcentral.org/budget for Dash.
