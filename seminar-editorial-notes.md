# Faculty seminar editorial notes

The deck has **26 main slides**, followed by a divider and **14 backup slides**. Prepared delivery is **31:10**, leaving about nine minutes for questions. The title is **Finding Regularity: High-Performance Computing for Graph Algorithms**.

## Revision scope

The latest opening revision applies review items 2–5. Slides 1–5 remain unchanged, including the introductory text on slides 1–3. Slide 6 now leads with the four sample counts and their concrete average, with priority-queue mechanics and general reachability formulas removed from the main slide. Slide 7 states the presenter's design decision explicitly. Slide 8 keeps the state-layout comparison and removes the hardware catalogue and unscoped latency/throughput figures. Slide 9 explains undirected component labels and connects their updates to the sample lanes. Slide 10 separately explains edge reconstruction and distinguishes repeatability from edge independence. The reconstruction mechanism now leads directly into its performance payoff on slide 11.

There are now 26 main slides. Every slide is static, and the compiled PDF has 41 pages including backups. Slides 9 and 10 share the original 1:45 allocation (0:50 and 0:55), preserving the 31:10 total and all later cumulative timings. The script and embedded cues follow the new order.

The previous revision reviewed every slide after slide 9. The main argument progresses from shared topology to state representation, sample scheduling, and dependent random-walk addresses. SABA remains the longest worked scheduling example. Ongoing projects and the future agenda occupy the final portion of the talk.

## What changed

| Slides | Revision | Purpose |
|---|---|---|
| 9 | Undirected minimum-label propagation, connected to the sample lanes | Explain the operation performed by each sample lane |
| 10 | Four stored random seeds and a traced XOR membership mask | Explain how each pass recovers the same active edges without storing each sampled graph |
| 11 | Stored-sample and fused-sampling execution diagrams beside the existing result | Show exactly which intermediate data movement is removed; seed sorting is introduced later with FASST |
| 12 | Vertex-by-sample state matrix and the reached set from slide 6 | Make the remaining state cost and the downstream count query concrete |
| 13 | Leading-zero example and a four-sample masked register merge | Explain the sketch's information, idempotence, and uniform machine operation |
| 14 | The same sample entries mapped to CPU lanes, GPU threads, and device partitions | Distinguish hardware mappings from sequential pipeline stages |
| 15 | Sample IDs remain visible before and after FASST assignment | Show what is reordered and why a whole warp can skip work |
| 16 | Existing scaling data paired with local propagation and global reduction | Explain the decomposition behind the reported result |
| 17 | Four walks request one, then three, then four adjacency rows | Expose dependent addresses and loss of exact alignment |
| 18 | The same eight random keys grouped into visible bouquets | Make the scheduling contribution traceable and define “bouquet” |
| 19 | Twelve inputs partitioned into equal intervals, with residual 7 → 3 highlighted | Explain conditional uniformity before presenting the full rule in backup |
| 20 | Existing cache-miss values shown as proportional bars | Connect direct vectorization, locality, and measured memory behavior |
| 21–22 | Machine cost / algorithmic change / effect, followed by explicit tradeoffs | Synthesize a methodology and motivate runtime decisions |
| 23–24 | Fused-walk pipeline, observed/missing diffusion states, and interleaved trajectory work | Give each ongoing project a concrete systems mechanism |
| 25–26 | Proposed observations and scheduling choices, evaluation criteria, and broader extensions | Establish a testable independent research program |

The backups retain implementation and correctness depth. The FASST backup now explains the actual key operation and device-local edge set instead of opaque set notation. Repeated paper-section footers were removed where unnecessary. The duplicate SABA measurement-scope slide was replaced by an explanation of when bouquet locality persists. The main performance slide and the benchmark backup retain the distinction between the reported earlier implementation and the corrected sampler.

## Technical boundaries retained

- Slide 9 uses undirected connected components; it does not equate minimum-label propagation with directed reachability. Slide 10’s membership mask matches edge one–four in the earlier example. Seeds are reused across all passes. Repeatable XOR decisions do not alone prove joint edge independence.

- The fused-sampling comparison supports on-demand sampling. It does not isolate the sample-lane transpose; batching is an additional transformation.
- Compact registers still occupy a vertex-by-sample structure. They change entry size, operations, and approximation rather than automatically reducing the dimensions of that structure.
- A sketch register is a rare-pattern statistic, not an exact cardinality. The main merge example leaves the inactive sample unchanged. Monte Carlo evaluation and error-adaptive rebuilding remain part of HyperFuseR.
- FASST sorts existing keys once and keeps their state associated with them. It exploits structure in the fused decision construction, not a generic guarantee of arbitrary hashing or a perfect per-edge sort.
- SABA's grouping example counts distinct requested adjacency rows, not physical cache misses. Later steps can diverge.
- The sampler explanation uses residual-range consumption and permanent unbiased fallback. Correctness is conditional on the labeled walk's history. It does not rely on the older fixed-seed/XOR argument or independence of sorted ranks.
- Distributional validity, estimator order invariance, and exact replay remain separate. The replay backup retains the requirement for walk-owned fallback randomness.
- NeuralBloom's roughly twofold result remains preliminary and scoped to walk computation. Battus and deterministic trajectories have distinct approximation requirements.
- No benchmark was rerun and no new empirical result was introduced. The bar chart uses the existing reported values.

## Assessment against the hiring-talk goal

| Question | Assessment after revision |
|---|---|
| Is the research question clear? | Each section answers how an algorithm can change the memory or scheduling cost already introduced. The ending returns to systematic architecture-aware choices. |
| Can a general CS audience follow? | State matrices, named samples, explicit masks, and small integer ranges replace several abstract summaries. New terms such as “bouquet” are defined where first used. |
| Is there technical depth? | Masked sketch propagation, sample/device decomposition, conditional residual uniformity, and explicit tradeoffs provide concentrated depth. Implementation and replay details remain available in backup. |
| Is the personal contribution visible? | The narration identifies the design decision in first person and uses reported joint results for evidence. It avoids implying ownership of standard sketch primitives. |
| Do the papers form one program? | Shared topology exposes state costs; sample ordering extends to devices; path-dependent walks force a new scheduling mechanism. |
| Does the talk suit an ML-heavy department seeking systems expertise? | The core remains algorithms, memory, and execution. NeuralBloom supplies a concrete current connection to graph learning, with a narrowly scoped result. |
| Is the future agenda independent and testable? | Adaptive scheduling has a hypothesis, observations, actions, baseline comparisons, and overhead-aware evaluation. Representation and CPU–GPU placement broaden the program. |
| Is there unnecessary implementation or benchmark detail? | Main results retain one purpose each. Proof details, intrinsics, extended comparisons, and sampler replay remain in backup. |

## Source map for preparation and questions

| Topic | Local source / existing provenance |
|---|---|
| Fused sampling, SIMD state layout | `papers/2008.03095v1-infuser-mg.pdf`, Sections 3.1–3.3 and Table 4 |
| Sketch construction, sample-aligned merge, rebuilding | `papers/2105.04023v1-hyperfuser.pdf`, Sections 2.2–3 and Algorithm 1 |
| FASST, device-local edges, distributed reductions, scaling | `papers/2410.14047v1-difuser.pdf`, Section 4.1, Algorithm 4, Table 8 |
| Corrected residual sampler and locality limits | `papers/aesc-revised-source/paper.tex`, Section 3.2 and locality discussion |
| Reported cache and timing comparison | Existing SABA Tables 5 and 7; historical implementation scope retained |
| NeuralBloom | `../neuralbloom/README.md`, `../neuralbloom/paper/main.tex`; preliminary result supplied by presenter |
| Battus | `../battus/paper/sections/method.tex` |
| Trajectory prototype | `../randomwalk/walk.h`, `../randomwalk/metal.h` |

Only source material needed to explain the algorithms was consulted. These projects' source files and experiments were not changed. All slide diagrams remain editable TikZ in `seminar.tex`; reference PDFs are not needed to build the deck.

## Rehearsal priorities

- **Slide 13, ending at 14:25:** distinguish a register value from a count, then trace the inactive lane through the merge. The planned 2:10 allows pointing and explanation.
- **Slide 15, ending at 17:05:** follow one sample identity across assignments. Explain why skipping a warp differs from eliminating the sample's contribution.
- **Slides 18–19, ending at 23:05:** first demonstrate the locality opportunity, then consume the residual range correctly. Keep these two arguments separate.
- **Slide 20, ending at 24:05:** state the measurement scope once and connect the bars to the address pattern. Avoid turning this into a benchmark inventory.
- **Slide 25, ending at 30:40:** describe the scheduling hypothesis as proposed work. State tuned static baselines, unseen workloads, total cost, and fixed accuracy as the evaluation criteria.

The revised narration for slides 11–26 ranges from approximately 110 to 138 words per minute at the assigned timings, leaving room for pointing at the technical diagrams. These are preparation targets, not a substitute for an aloud rehearsal. If questions consume two minutes, shorten the synthesis and current-project details before cutting the worked sketch or residual-range examples.

## Build and verification

Use the build commands in `README.md`. The prepared PDF is `seminar.pdf`; the narration is `seminar-narrative.md`. Compact delivery cues remain embedded in the LaTeX source.
