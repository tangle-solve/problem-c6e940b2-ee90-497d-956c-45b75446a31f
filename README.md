<!-- solve-problem:c6e940b2-ee90-497d-956c-45b75446a31f -->
# Rehypothecation-aware yield aggregation

A [Tangle Solve problem](https://solve.tangle.tools/problems/c6e940b2-ee90-497d-956c-45b75446a31f).

## What is the problem?

In decentralized finance (DeFi), collateral circulates through nested lending and staking structures: a user deposits ETH into lending protocol A, receives a receipt token (e.g., a wrapped aToken/cToken), deposits that into vault B, receives another receipt, and so on. This is on-chain rehypothecation — the same underlying asset backs multiple layers of claims. The task is to design an accounting/auditing mechanism (smart-contract-level or network-level) that computes, for every protocol in a dependency graph, its **ultimate collateral backing** and **contingent liabilities**, resolving chains of claims down to primitive assets, without double-counting diversification benefits when the same underlying asset appears via multiple paths.

Formalize as a directed graph G = (V, E): nodes are protocols/vaults/tokens, edges are collateral relationships with weights (claim amounts, collateral factors, liquidation thresholds). Given a shock (price move or default of node v), compute propagated losses and solvency of each node. The core difficulty: naive path-enumeration double-counts shared collateral; naive aggregation ignores path-dependent liquidation cascades. The goal is a sound consolidated-balance-sheet computation, analogous to consolidated accounting in traditional finance, executable on-chain or by auditors, with provable consistency (sum of net exposures equals actual underlying).

The "engineering" type flag means the deliverable may be a mechanism plus implementation, not purely a theorem.

## Why does it matter?

Currently, no DeFi analytics tool or protocol natively tracks ultimate exposure through receipt-token chains, so systemic risk is invisible: protocols appear diversified while actually sharing the same collateral. A solution would let risk managers, auditors, and regulators assess true leverage and contagion depth before a cascade (cf. historical DeFi liquidation spirals), and would enable vaults to price or cap nested borrowing against already-encumbered collateral. Without it, composable credit keeps amplifying hidden correlated exposures that only surface during stress events.

## What's already known?

- Werner et al. (2022) provide the SoK taxonomy of DeFi, including lending, composability, and rehypothecation patterns — descriptive, no formal accounting mechanism.
- Tovanich et al. (2023) empirically simulate liquidation contagion on Compound, but treat a single protocol layer, not nested receipt-token chains.
- Prior contagion literature in traditional finance (Eisenberg–Noe style clearing models) gives fixed-point cascade solvers for interbank networks, but does not handle tokenized claims or on-chain graph extraction.
- Heuristic tools (DeBank, Zerion, DefiLlama) aggregate TVL per protocol and are known to double-count wrapped/nested deposits — this is precisely the double-counting problem, unsolved in production analytics.
- Consolidated accounting offers a conceptual template (look-through, entity-level netting) but no algorithmic treatment of cyclic token graphs.

No published mechanism computes ultimate collateral for general cyclic DeFi dependency graphs with soundness guarantees.

## What would count as solving it?

1. **Specification + algorithm:** a formally defined exposure graph and an algorithm computing ultimate-collateral vectors and contingent liabilities for every node, with a proof of (a) conservation (no double-counting; net claims telescope to primitive assets) and (b) correctness under liquidation shocks (matching the cascade fixed point or bounding the discrepancy).
2. **Implementation:** on-chain or indexer-based prototype evaluated on real receipt-token data (e.g., Aave→Yearn→Curve nesting patterns), demonstrating it flags exposures that TVL aggregation overstates.
3. **Extension:** a mechanism whereby vaults enforce exposure caps using the computed quantities (rehypothecation limits), with game-theoretic analysis of circumvention incentives.

Any one of (1)–(2) with empirical validation would count; (3) is the stronger version.

## How to start

1. Read Werner et al. (2022) §on lending and composability to enumerate actual nesting patterns (which receipt tokens are re-deposited where in practice).
2. Model the system as a weighted directed graph; adapt Eisenberg–Noe-style clearing to tokenized claims; note the new twist: cyclic ownership (vault A holds vault B's token, B holds A's) means the liability matrix is not triangular — compute via fixed point and check convergence conditions.
3. Extract a real subgraph from on-chain data (Aave v2/v3, Yearn, stETH derivatives) and test whether naive summation double-counts; quantify the gap.
4. Main obstacle: liquidation thresholds and oracle dynamics are path-dependent, so "ultimate collateral" under stress may not be a static graph quantity — consider bounding rather than exact computation.

## Key references

- Werner, S. M., Perez, D., Gudgeon, L., Klages-Mundt, A., Harz, D., & Knottenbelt, W. J. (2022). "SoK: Decentralized Finance (DeFi)." ACM Computing Surveys. arXiv:2101.08778. Taxonomy of DeFi including lending, composability, and rehypothecation risks — the definitional starting point.
- Tovanich, N., He, N., Li, R., Cai, Y., & Knottenbelt, W. J. (2023). "Contagion in Decentralized Lending Protocols: A Case Study of Compound." DOI:10.1145/3605768.3623544. Empirical agent-based simulation of liquidation cascades on Compound; the closest existing contagion methodology, single-layer.
- Eisenberg, L., & Noe, T. H. (2001). "Systemic Risk in Financial Systems." Management Science 47(2). Classic clearing-vector fixed-point model for interbank contagion — the natural mathematical substrate to extend. (Standard reference; verify applicability to cyclic token graphs.)

## Warnings

- The problem statement says "engineering," but a referee will still demand a well-defined semantics of "ultimate collateral" under stress; define it before building.
- Double-counting is not just TVL inflation — diversification double-counting arises when a protocol holds two different receipt tokens backed by the same asset and treats them as independent.
- Cycles in the token graph (A holds B, B holds A) break naive recursive resolution; you need fixed-point or truncation semantics.
- On-chain enforcement is much harder than off-chain auditing; scope accordingly.
- Live protocol data is messy: token wrappers, rebasing vs. non-rebasing receipts, and deprecated vaults complicate graph extraction.

---
*`BCGT-000349` · flagged for review*

## Acceptance guidance

1. **Specification + algorithm:** a formally defined exposure graph and an algorithm computing ultimate-collateral vectors and contingent liabilities for every node, with a proof of (a) conservation (no double-counting; net claims telescope to primitive assets) and (b) correctness under liquidation shocks (matching the cascade fixed point or bounding the discrepancy).
2. **Implementation:** on-chain or indexer-based prototype evaluated on real receipt-token data (e.g., Aave→Yearn→Curve nesting patterns), demonstrating it flags exposures that TVL aggregation overstates.
3. **Extension:** a mechanism whereby vaults enforce exposure caps using the computed quantities (rehypothecation limits), with game-theoretic analysis of circumvention incentives.

Any one of (1)–(2) with empirical validation would count; (3) is the stronger version.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/c6e940b2-ee90-497d-956c-45b75446a31f?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000349.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000349)
