# Boundless Prover Guide

## Boundless Prover market
First, you need to know how **Boundless Prover market** actually works to realize what you are doing.
* **Requester Submits Ask**: A requester (e.g. developer) creates a task or a computation request as an `order` on Boundless, offering funds in ETH or ERC-20 to incentivize participation.
* **Prover Stakes USDC**: The Boundless market requires funds (USDC) deposited as stake before a prover can `bid` on requests.
* **Prover Places Bid**: A prover detects an `order`, submits a `bid`, stating their offered price or resources, which may be lower than the request’s locked funds or other provers’ `bid`s.
* **Prover Locks Order**: If their `bid` is accepted among other provers (e.g., lower `bid`, sufficient stake, or meeting specific criteria), the prover locks the order committing to prove it within a set deadline (`lock-timeout`) using previously staked `USDC`, so other provers can't touch it until it perform computational power.
* **Prover Generates Proof**: The prover completes the task and submits the `proof` to the network.
* **Slashing**: If the `proof` is invalid, incomplete, or the prover fails to deliver (e.g., due to low computational power, malicious behavior or timeout), the slashing mechanism activates, penalizing the prover by forfeiting a part of their staked `USDC` funds.
* **Order Fulfillment**: If the `proof` is valid, the prover receives the locked funds as a reward, and the requester receives the verified result, completing the process.
* `bid ` are actually `mcycle_price` (the price of each 1 million cycles the prover proves). I'll tell you more about this later in the guide.

---

## Notes
- The prover is in beta phase, while I admit that my guide is really perfect, you may get some troubles in the process of running it, so you can wait until official incentivized testnet with more stable network and more updates to this guide, or start exprimenting now.
- I advice to start with testnet networks due to loss of stake funds
- I will update this github guide constantly, so you always have to check back here later and follow me on [X](https://x.com/0xMoei) for new updates.

---
