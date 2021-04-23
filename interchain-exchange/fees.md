# Interchain Exchange fees

## Game theory exploits

The following fees are suggested to prevent the following:

1. Users could potentially take up their own orders to generate fake transactions.
1. Users could potentially take up other users' orders to eliminate competition and increase own incentives returns.
1. Users could leave their orders passively and not respond to take requests.

## General ideas

- Negative maker fee
- Positive taker fee
- Also the DFI made by Maker, inclusive of bonus must be less than fees paid by Taker. Without it fake transactions would be generated to game rewards.

## Fees

- Previous `optionFee` is now replaced with non-refundable `takerFee`.
    - This is to deal the case where Alice refuses to accept Bob's valid HTLC _(Scenario 1)_ and unfairly profits from `optionFee`.
- `takerFee` is in DFI and is fixed at consensus-set (`takerFeePerBTC` * BTC * `DEX DFI per BTC rate`).
- `makerDeposit` being the same as `takerFee`.
    - It is refundable upon successful swap.
- `makerIncentive`
    - `25% * takerFee`
- `makerBonus`
    - `50% * takerFee` if it's BTC parity trade. BTC DSTs eligible for `makerBonus` are consensus-set.


### DST maker exiting to Bitcoin

1. Maker, pays no fee, but locks up DST _(consensus-verifiable)_.
2. Taker makes offer, locks up `takerFee`.
    - If offer is not accepted after timeout, `takerFee` is refunded.
3. Maker accepts offer, locks up DST _(consensus-verifiable)_, burns `makerDeposit` & `takerFee`.
    - If partial amount from offer is accepted, refunds unused `takerFee` proportionally back to Taker.
4. Taker locks up BTC _(non-consensus-verifiable)_ and updates ICXC on BTC HTLC.
5. Maker verifies that BTC HTLC is correct, claims BTC.
    - If Taker locks up incorrectly, both parties lose DFI _(neither party wins)_
    - If Maker does not verify, both parties lose DFI _(neither party wins)_
6. Taker claims DST on ICXC.
    - Mints `makerDeposit` _(refund)_, `makerIncentive` & `makerBonus` _(if any)_ to Maker.
    - Validate that the amounts are correct – no additional tokens are generated.

### BTC maker entering from Bitcoin

1. Maker, pays no fee.
2. Taker makes offer, locks up `takerFee`.
    - If offer is not accepted after timeout, `takerFee` is refunded.
3. Maker accepts offer, locks up `makerDeposit`.
4. Taker locks up DST _(consensus-verifiable)_, burns both `takerFee` and `makerDeposit`.
    - If Taker does not do it, burns `takerFee` and refund `makerDeposit` to Maker.
5. Maker locks up BTC _(non-consensus-verifiable)_ and updates ICXC on BTC HTLC.
6. Taker verifies that BTC HTLC is correct, claims BTC.
    - If Maker locks up incorrectly, both parties lose DFI _(neither party wins)_
    - If Taker does not verify, both parties lose DFI _(neither party wins)_
7. Maker claims DST on ICXC
    - Mints `makerDeposit` _(refund)_, `makerIncentive` & `makerBonus` _(if any)_ to Maker.
    - Validate that the amounts are correct – no additional tokens are generated.
