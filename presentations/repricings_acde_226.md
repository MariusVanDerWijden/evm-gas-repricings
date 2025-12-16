---
title: Glamsterdam Repricings - ACDE \#222
---

<style>
.reveal {
  font-size: 36px;
}
</style>

# Glamsterdam Repricings

## ACDE \#226

### Maria and Marius

#### December 18th, 2025

[Slides](https://notes.ethereum.org/@misilva/S15O_Cpzbg)

---

## Agenda

- Update on the status of the core repricings by Maria
- A proposal for a pragmatic way forward with repricings by Marius

---

## Core Repricings

A broad harmonization of burst resources:

- Compute (EIP-7904, EIP-2780)
- State Access (EIP-8038, EIP-2780)
- Data (EIP-7976, EIP-7981)

Plus, attempting to mitigate state growth under higher block limits (EIP-8037)

---

## Compute :computer:

- Numbers on EIP-7904 are not finalized yet
  - Missing precompiles
  - Some numbers introduce new bottlenecks based on the most recent testing
  - Conversion between execution times and gas prices didn't consider gas limit target
- Numbers on EIP-2780 will be updated based on EIP-7904

---

## State Access :floppy_disk:

- Numbers on EIP-8038 are still missing
  - First-level benchmarks will likely lead to an increase in cost
  - However, after BALs optimization, the cost will likely be lower
- Numbers on EIP-2780 will be updated based on EIP-8038

**Need to get client optimizations ASAP, so we can benchmark them!**

---

## Data :globe_with_meridians:

- Numbers on EIP-7976 and EIP-7981 are likely to not change much
- However, we will need to double-check it after we decide on the payload PTC deadline in ePBS

---

### 🎁

### Wen final numbers?

---

## Benchmarking update

- Nethermind, STEEL and Prototyping teams are working tirelessly to get us execution times for all opcodes
- Maria will analyze data and propose new numbers
- Plan to get the numbers without BALs optimizations by the next ACDE

---

### 🤔

### Can we move forward without final numbers?

---

## A pragmatic way forward

1. Align on the gas limit for Glamsterdam
2. Increase the price of all operations performing worse than the gas target (after BALs and ePBS)
   - These are the bottlenecks to reach the gas target, so they need to be repriced
3. Decrease the price of some simple compute operations
   - This will increase the throughput for these operations
   - We will not reprice every operation, only the ones that are simple to test and that are frequently used
4. Figure out how to handle state growth

---

## What gas limit to target?

- Trade-off between throughput and number of mispriced operations:
  - Higher gas limit means more operations to increase the price of
  - Lower gas limit means lower throughput on all other operations

---

**Suggestion: 100M gas limit before ePBS**

This means:

- Repricing all operations performing below 35M gas/second after BALs
  - 15 operations according to gas bench data from Dec. 15th
  - CALL*, some EXTCODE*, point eval, some simple arithmetic
- Reaching ~300M-200M gas limit with ePBS

---

## What about state growth?

1. Align a target state growth rate
   - This will inform how aggressive we need to be
   - Three angles to consider

---

### Three angles to consider

- State I/O performance.
  - Likely the most critical.
  - Stateful benchmarks will give us better numbers on the current status and on how performance degrades from mainnet to bloatnet.
  - BALs very likely to improve this significantly!
- Sync/healing.
  - This is not a concern now, but we don't yet know when it will become an issue.
  - Carlos is bloating again to allow us to test this under bigger states.
- Block building.
  - Likely not a priority for now, but we should start benchmarking this

---

## What about state growth?

1. Align a target state growth rate
   - This will inform how aggressive we need to be
   - Three angles to consider
2. Select the approach for EIP-8037
   - Three options:
     - No action (current state)
     - Lukasz's dynamic pricing
     - Anders' EIP-8075

---

### Thank you

### :cat2: