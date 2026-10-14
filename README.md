<!-- solve-problem:165655cd-c27b-4700-8164-3ea4b9db1f07 -->
# Hundredfold relay-efficiency frontier

A [Tangle Solve problem](https://solve.tangle.tools/problems/165655cd-c27b-4700-8164-3ea4b9db1f07).

## What is the problem?

The problem asks whether a blockchain relay/propagation protocol can achieve roughly a **100× improvement** in throughput or communication efficiency over a baseline (e.g., Bitcoin Core's block relay) **on fixed consumer hardware** (ordinary CPUs, home bandwidth), while **retaining explicit adversarial security guarantees** — i.e., provable correctness/liveness against Byzantine adversaries and network-level attacks (eclipse, censorship, DoS amplification).

Formal setting: a peer-to-peer network of n nodes, an adversary controlling some fraction of nodes/bandwidth, a broadcast primitive for blocks. Metrics: (a) blocks/sec the network can propagate, (b) bytes transmitted per block relayed (comm. complexity), (c) latency until a honest node accepts a block. Baseline: standard relay sends full blocks (or BIP 152 compact blocks: header + short IDs + missing transactions, roughly 10–90× reduction vs. naive relay in the best case, but with no formal adversarial analysis and limited by transaction-mempool hit rate). The target: a protocol achieving ~100× (either throughput scaling or per-block communication reduction) with a stated theorem bounding the adversary's influence. The exact baseline and threat model are **unspecified in the original statement** — a solver must fix these (see Warnings).

## Why does it matter?

Block propagation bandwidth is the binding constraint on on-chain throughput: larger blocks mean orphan risk and centralization, so relay efficiency directly caps safe block size. A provably secure 100× relay improvement would let existing chains scale two orders of magnitude without hardware centralization, and would make scaling claims comparable across projects via a common adversarial yardstick. It also connects to fundamental limits: information-theoretic broadcast capacity and communication-complexity lower bounds tell us how much compression relay can achieve at all.

## What's already known?

- **BIP 152 compact block relay** (Corallo, 2016): sends an 80-byte header plus 6-byte short transaction IDs; peers reconstruct from mempool and request only missing transactions. In practice large savings, but heuristic: relies on mempool overlap, no adversarial proof; adversaries can force worst-case fallback (full block resend) via ID collisions or withholding.
- **Prism** (Bagaria et al., 2019): decouples transaction blocks from proposer blocks; achieves throughput approaching physical network limits with provable security under the "locally predictable" propagation model. Prism's security proofs assume honest majority of stake and synchronous propagation, but Prism targets consensus-layer throughput, not relay-layer communication compression; the two axes have not been combined.
- **No known construction** simultaneously achieves: (i) ~100× measured efficiency gain on commodity hardware, (ii) formal adversarial guarantees for the relay layer itself. Lower bounds on broadcast communication complexity are known in the Byzantine broadcast literature (e.g., Ω(n²) message complexity results, Dolev–Reischuk-style), suggesting naive approaches can't scale; whether compression to near-entropy is possible under attack is open.

## What would count as solving it?

Any of the following would constitute meaningful progress; the full moonshot requires (1) + (2):

1. **A protocol + implementation** that on fixed consumer hardware (specify CPU, bandwidth, e.g., a 2018 laptop on 50 Mbps) demonstrates ≥100× aggregate throughput or ≥100× per-block communication reduction versus a stated baseline (e.g., full-block relay or BIP 152 at defined mempool hit rates), **and**
2. **A theorem** in a stated adversary model (fraction f of Byzantine nodes, network timing assumptions) bounding: false accepts, liveness latency inflation, and communication blow-up caused by the adversary.
3. Partial results: a lower bound showing 100× is information-theoretically impossible under a realistic model (negative resolution), or a 10× construction with full proofs (progress milestone).

A referee would want: the baseline, hardware, and threat model stated up front; measurements from a testnet or simulation with adversarial nodes; and the proof.

## How to start

1. **Read BIP 152** (short spec) to fix the practical baseline, then **Section on security model of Prism** (arXiv:1810.08092, esp. their propagation/locally-predictable assumption) — identify where each falls short of the combined target.
2. **Formalize the relay problem** as a Byzantine broadcast with correlated side information (mempools): nodes share most transactions; relay is essentially source coding with Byzantine encoders. Study set-reconciliation literature (e.g.,Graphene-style Bloom-filter reconciliation, Minisketch/CPisync) — error-correcting set reconciliation may give provable reconstruction under malicious inconsistencies.
3. **Derive the information-theoretic limit**: given honest mempool overlap α, the minimum communication to identify a block's transaction set is roughly H(membership | mempool) — check whether adversaries can inflate this unboundedly (they can inject transactions), and design rate-limiting/vouching countermeasures.
4. **Main obstacle**: adversarial mempool divergence breaks compression assumptions; naive authentication requires the data you're trying to avoid sending.

## Key references

- Corallo, M. (2016). *BIP 152: Compact Block Relay.* https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki — Defines short-ID compact blocks; the de facto practical baseline for relay efficiency; no adversarial analysis.
- Bagaria, N., Das, S., Tse, S., Xue, F., Fanti, G. (2019). *Deconstructing the Blockchain to Approach Physical Limits.* arXiv:1810.08092 — Proposes Prism, proving throughput approaching network physical limits under a propagation model; the security framework to extend to the relay layer.
- (verify) Ozisik & Andresen (2017), *Graphene: Efficient Interactive Set Reconciliation Applied to Blockchain Propagation* (CACM) — Bloom-filter/IBLT set reconciliation for blocks; candidate mechanism with failure-rate analysis but not adversarial.
- (verify) Dolev, D., Reischuk, R. (1985), *Bounds on Information Exchange for Byzantine Agreement*, JACM — Lower bounds on communication for Byzantine agreement; relevant to fundamental limits.

## Warnings

- The problem statement is vague: "hundredfold" versus what baseline, and over what metric (bandwidth per block? blocks/sec? latency-adjusted throughput?) must be pinned down before proving anything — a paper can trivially claim 100× against a strawman baseline.
- "Retaining adversarial guarantees" is undefined; you must specify adversary power (fraction of nodes, bandwidth, ability to withhold/collide short IDs). BIP 152 is known to degrade adversarially via ID collisions and mempool manipulation.
- Simulation-only results with no threat model won't satisfy the "explicit adversarial guarantees" clause; likewise, pure asymptotics without a hardware demo miss the "fixed consumer hardware" clause.
- Beware double counting: Prism already claims near-physical-limit throughput; clarify that its gain is consensus-architecture-level, not relay-communication-level, so the moonshot is about the relay layer or the honest combination of both.

---
*`BCGT-000276` · flagged for review*

## Acceptance guidance

Any of the following would constitute meaningful progress; the full moonshot requires (1) + (2):

1. **A protocol + implementation** that on fixed consumer hardware (specify CPU, bandwidth, e.g., a 2018 laptop on 50 Mbps) demonstrates ≥100× aggregate throughput or ≥100× per-block communication reduction versus a stated baseline (e.g., full-block relay or BIP 152 at defined mempool hit rates), **and**
2. **A theorem** in a stated adversary model (fraction f of Byzantine nodes, network timing assumptions) bounding: false accepts, liveness latency inflation, and communication blow-up caused by the adversary.
3. Partial results: a lower bound showing 100× is information-theoretically impossible under a realistic model (negative resolution), or a 10× construction with full proofs (progress milestone).

A referee would want: the baseline, hardware, and threat model stated up front; measurements from a testnet or simulation with adversarial nodes; and the proof.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/165655cd-c27b-4700-8164-3ea4b9db1f07?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000276.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000276)
