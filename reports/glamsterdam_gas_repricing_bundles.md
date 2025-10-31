# Gas Repricings for Glamsterdam

### Maria Silva, October 2025

In this document, we provide an overview of all the repricing EIPs proposed for Glamsterdam, suggest bundle options and discuss how to prioritize them.

## Why now?

A key focus for the Glamsterdam fork is scaling the L1, with both headliners contributing to that goal:

- [EIP-7732](https://eips.ethereum.org/EIPS/eip-7732): Enshrined Proposer-Builder Separation (ePBS)
- [EIP-7928](https://eips.ethereum.org/EIPS/eip-7928): Block-Level Access Lists (BALs)

In this context, gas repricings play two key roles in L1 scalability:

1. They directly impact scaling through harmonisation.
2. They unlock latent scaling in both headliners.

On the first point, we know that today EVM operations don't have the exact costs for the same resource usage. For example:
- Compute operations do no spent the same MGas/s → [Opcodes Benchmarking](https://grafana.observability.ethpandaops.io/d/feo4ronhsqv40d/opcodes-benchmarking?orgId=1&from=now-30d&to=now&timezone=browser&var-posgreSQL=benuragv7iuwwb&var-ClientName=besu&var-TestTitle=EcRecover%20precompile&var-TestTitle=EcRecoverCACHABLE&var-TestTitle=EcRecoverUNCACHABLE&var-TestTitle=EcRecoverUNCACHABLE2&refresh=auto)
- State creation does not have the cost, depending on the operation → [Bloating Technique Ranking](https://cperezz.github.io/bloatnet-website/bloating.html)
- Data from access lists is free (i.e., users only pay for state access and growth)

With broad repricings across all EVM operations, we can fix these "pricing bugs" and get a one-time improvement in throughput. Instead of having a single operation set the maximum gas limit for the chain, we get harmonised performance across the board and thus more space for the currently overpriced transactions.

On the second point, the headliners will significantly change the relative costs of different EVM resources:

- ePBS will give more time in the slot for execution and data propagation. Also, there will be more time for execution than for data propagation, given the PTC deadline.
- BALs allow parallel execution and state root calculation but incur a data cost for state operations.

Thus, the relative cost of state growth to burst resources such as compute, data, and state access must increase, **and** the relative cost of data to compute and state access must increase. Without repricings, state growth and data will become bottlenecks to scaling, reducing the throughput we could otherwise achieve in compute and state access operations.

Caspar and Ansgar make similiar arguments in their [opinion piece](https://notes.ethereum.org/hdqymropR6eFMSXLoz_vTQ).

## EIPs and bundle options

There are three types of EIPs within the repricing scope:

1. **Broad harmonisation**. These EIPs reprice a class of operations to harmonize them and remove single bottlenecks.
2. **Pricing extension**. These EIPs make targeted changes to a specific opcode or component of the gas model, usually coupled with a new mechanism.
3. **Supporting**. These EIPs are not directly performing repricing, but instead introduce a change that supports other repricing EIPs or enhances the scalability potential of repricing.

For the complete list of EIPs, refer to the [Meta EIP-8007](https://eips.ethereum.org/EIPS/eip-8007). Next, we discuss options for bundling these EIPs.

### Minimal Repricing Bundle

The Minimal Repricing includes a broad repricing of compute and state operations, thus fixing the majority of mispriced opcodes, aligning the cost of ETH transfers with the relevant compute and state operations, and updating data costs. 

It is important to stress that this minimal set of EIPs must be done together. If we leave specific resources untouched while repricing others, we will introduce new sources of imbalance and bottlenecks into our pricing model.

Specifically, it:

- Harmonises the costs of compute operations with [EIP-7904](https://eips.ethereum.org/EIPS/eip-7904), thus removing single bottlenecks and allowing a higher throughput of compute operations.
- Updates the costs of state access operations with [EIP-8038](https://eips.ethereum.org/EIPS/eip-8038), thus removing state access as a scaling bottleneck.
- Increases the cost of state creation operations with [EIP-8037](https://eips.ethereum.org/EIPS/eip-8037), thus allowing for higher transaction throughput without accelerating state growth.
- Aligns the cost of ETH transfers with the rest of compute and state operations with [EIP-2780](./eip-2780), thus increasing the throughput of ETH transfers and aligning their cost with similar operations.
- Charges access lists for their data footprint with [EIP-7981](https://eips.ethereum.org/EIPS/eip-7981), thus lowering the worst-case block size achieved through call data.
- Increases call data cost for data-heavy transactions with [EIP-7976](https://eips.ethereum.org/EIPS/eip-7976), thus lowering the worst-case block size achieved through call data.

This is the minimum viable bundle that allows us to remove single bottlenecks while leveraging the gains obtained from ePBS and BALs.

On the one hand, we are doing a broad repricing of key burst resources (compute, state access, and data) that will take into account the relative performance of operations within individual resources, as well as changes in available execution time and data propagation delivered by ePBS and BALs.

On the other hand, we are increasing the cost of state growth to ensure its growth rate does not exceed unfeasible levels, given the increased transaction throughput. State growth is the most pressing of the persistent resources to address, as it affects hardware requirements and state access (and thus execution time). Additionally, there is no clear path yet to a long-term solution for state growth, which makes mitigating it even more critical.

### Recommended Additions

The Extended Repricing includes all changes from the Minimal Repricing, plus some pricing extensions that mitigate worst-case blocks or enhance throughput on the average case. 

In contrats with the minimal pricing, this collection of EIPs can be independently included...


The additions include:

- Excluding refunds from block accounting with [EIP-7778](https://eips.ethereum.org/EIPS/eip-7778), thus mitigating the worst-case block from state access operations. This should be a simple change that tackles a concrete bottleneck.
- Scaling the cost of `SSTORE` with the contract's storage size with [EIP-8032](https://eips.ethereum.org/EIPS/eip-8032), thus allowing smaller contracts to write to their storage more cheaply, while solving the state access bottleneck of large contracts. Compared with [EIP-8038](https://eips.ethereum.org/EIPS/eip-8038), this change improves the throughput of `SSTORE` operations on all contracts smaller than ~8 GB of data by making them cheaper. Even though this EIP includes a change to the trie, given that state access is a key bottleneck, we expect the gains from this change to outweigh the added complexity. However, more benchmarking is needed to gather final numbers.
- Decreasing the costs of `TLOAD` and `TSTORE` with [EIP-7971](https://eips.ethereum.org/EIPS/eip-7971), thus improving the usability of transient storage. Pushing more users to leverage transient storage will ease the burden on state access.
- Solving rounding errors from efficient compute operations with [EIP-8053](https://eips.ethereum.org/EIPS/eip-8053) or [EIP-8059](https://eips.ethereum.org/EIPS/eip-8059). On average, transactions are expected to incur 4% rounding error in compute operations.
- Reduce the cost for deploying duplicated contracts with [EIP-8058](https://eips.ethereum.org/EIPS/eip-8058).

### Possible Additions

The following EIPs tackle some mispriced operations. However, they also introduce new mechanisms that warrant further consideration.

- Add warm account writes with [EIP-7973](https://eips.ethereum.org/EIPS/eip-7973).
- Change the memory expansion cost from quadratic to linear with [EIP-7923](https://eips.ethereum.org/EIPS/eip-7923).
- Add code chunking and update code deployment cost with [EIP-2926](https://eips.ethereum.org/EIPS/eip-2926).
- Add Multidimensional Gas Metering with [EIP-8011](https://eips.ethereum.org/EIPS/eip-8011).
- Add multi-block state access discounts with [EIP-8057](https://eips.ethereum.org/EIPS/eip-8057).



