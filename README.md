# 🧩Boundless Prover Guide - The Most Detailed One
The Boundless Prover Node is a computational proving system that participates in the Boundless decentralized proving market. Provers stake USDC bid on computational tasks generate zero-knowledge proofs using GPU acceleration and earn rewards for successful proof generation.

---

## First under how provers work - to know what you are doing : don't skip any line

- Requester posts ZKP request on Boundless (say 10M cycle proof in $5-$10 and 30 min is deadline)
- All provers (say Prover 1-5) see request simultaneously
- Prover 1 locks order at $5 he has to fulfill in 10 mins (LockAndFulfill order)
- Provers 2-5 skip locked order awaiting higher price
  * If Prover 1 attempts proof `Success` Submits in 10 mins, gets paid, no slashing
  * If Failure: Lock expires, Prover 1 slashed

- Post-lock expiry: Order becomes FulfillAfterLockExpire; secondary provers (2-5) attempt proof in remaining 20 mins.
- First secondary prover to submit wins payment; others earn nothing, no slashing.

---

- In all this chaos Boundless has mentioned rewards for primary provers - but i did not come across a single post which says Secondary Provers will be rewarded too - maybe in future they may announce
- Almost all one click nodes (mintair, easy nodes etc) are not able to lock orders even if they do are max secondary orders

---

## Requirements
### Hardware
* CPU - 16 threads, reasonable single core boost performance (>3Ghz)
* Memory - 32 GB
* Disk - 100 GB NVME/SSD
* GPU
  * Minimum: one 8GB vRAM GPU
  * Recommended to be competetive: 10x GPU with min 8GB vRAM
  * Recomended GPU models are 4090, 5090 and L4

### Software
* Supported: Ubuntu 20.04/22.04
* Not supports : Ubuntu 24.04

---
