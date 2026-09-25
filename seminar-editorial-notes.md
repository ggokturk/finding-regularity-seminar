# Faculty seminar editorial notes

The deck has **25 main slides**, followed by a divider and **16 backup slides**: **42 static PDF pages**. The title is **Finding Regularity: High-Performance Computing for Graph Algorithms**.

Prepared narration totals **30:20**. Reserve **35 minutes** for delivery with pauses and brief interruptions, with **40 minutes as the ceiling** in the one-hour seminar and interview session. This leaves roughly 20–25 minutes for discussion and interview questions. Timings are rehearsal targets, not measured delivery times.

## Opening revision for an ML faculty audience

The opening now follows a continuous argument: irregular memory access limits useful parallelism; shared topology creates a sample dimension to organize; adjacent sample state improves the access pattern; reconstructing the same edge decisions avoids materialized sampled graphs; the fused-sampling result supplies the payoff.

The former slides 9 and 10 are replaced by one main slide, **The same sampled edges on every pass**. Its left side explains the retained sample key and edge hash; its right side shows the same active mask for edge 1 to 4 on successive passes. Evolving vertex state and fixed graph membership are distinguished before discussing the memory saving. The component-label and numerical XOR examples are preserved as the first two backup slides.

This avoids switching the main explanation from directed reachability to an undirected component algorithm before the payoff. The undirected component-label implementation is introduced briefly as an example of exact state on slide 11, with its full update available in backup. The merged slide illustrates repeatability; it does not claim that repeatability establishes the required joint sampling distribution.

| Slides | Editorial decision | Delivery purpose |
|---|---|---|
| 1–2 | Keep title and background brief | Establish research identity within the first minute |
| 3 | Shorten the HPC overview to 30 seconds | Preview the arithmetic versus data-movement tradeoff |
| 4 | Keep the lane diagram and trace only selected lanes | Establish memory stalls and unequal useful work |
| 5 | Add a spoken connection to current graph-learning work | Explain the relevance to the faculty audience early |
| 6 | Explain one sample, then point to all four counts and their average | Establish repeated stochastic graph traversal without an edge-by-edge tour |
| 7–8 | Retain shared topology and the column/row access comparison | Make the presenter's execution decision concrete |
| 9 | Combine repeated propagation and reconstructed edge membership | Connect sample lanes directly to the avoided sampled-graph storage |
| 10 | Keep the fused-sampling pipeline and scoped result | Reach the first quantitative payoff at 10:40 |
| 11 | Briefly define component labels as an example of retained state | Connect memory savings to the remaining state bottleneck |
| 12–25 | Renumber and update all cumulative timings | Preserve the remainder of the research narrative |

The main narration, timing table, source comments, and embedded speaker cues use the same numbering and time allocations. Slides remain static.

## Technical boundaries retained

- Main slide 9 follows the directed running example only to illustrate repeated edge membership. The backup component-label algorithm applies to undirected graphs, where component sizes support influence estimates; several seeds count each component once.
- The backup XOR construction uses a symmetric hash for undirected edges. Reusing keys establishes repeatability across passes and directions, not joint edge independence.
- The fused-sampling comparison supports on-demand sampling. Samples execute separately in that comparison; it does not isolate the sample-lane transformation.
- Compact registers still occupy a vertex-by-sample structure. They change entry size, operations, and approximation rather than removing that structure's dimensions.
- A sketch register is a rare-pattern statistic, not an exact cardinality. Monte Carlo evaluation and error-adaptive rebuilding remain part of HyperFuseR.
- FASST sorts existing keys once and keeps their state associated with them. Its grouping exploits the actual fused decision construction, not a generic property of arbitrary hashing.
- SABA's grouping example counts distinct requested adjacency rows, not physical memory transactions. Later steps can diverge.
- The corrected sampler uses residual-range consumption and permanent unbiased fallback. Validity is conditional on the labeled walk's history; sorting alone cannot repair an incorrect distribution.
- The SABA cache and timing results concern the earlier implementation, not measurements of the corrected residual sampler.
- NeuralBloom's roughly twofold result remains preliminary and limited to walk computation. Battus and deterministic trajectories have distinct approximation requirements.
- No benchmark was rerun and no empirical claim was added.

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

- **Slide 8, ending at 8:10:** slowly trace the scattered column and adjacent row. Keep the distinction between logical state accesses and physical memory transactions.
- **Slide 9, ending at 9:25:** distinguish changing vertex state from fixed edge decisions; trace only edge 1 to 4. Keep the component update and XOR arithmetic for questions.
- **Slide 10, ending at 10:40:** identify the stored graph write/read that fusion removes, then state the result and its sequential comparison scope.
- **Slide 12, ending at 13:35:** distinguish a register value from a count, then trace the inactive lane through the merge.
- **Slide 14, ending at 16:15:** follow one sample identity across assignments. Explain why skipping a warp differs from removing a sample's contribution.
- **Slides 17–18, ending at 22:15:** demonstrate locality opportunity, then explain conditional residual uniformity.
- **Slide 19, ending at 23:15:** state the historical measurement scope once.
- **Slide 24, ending at 29:50:** present the scheduling hypothesis as proposed work, with tuned baselines, unseen workloads, total cost, and fixed accuracy as evaluation criteria.

The 35-minute delivery slot includes about four and a half minutes beyond the prepared narration. Rehearse aloud with pointing and pauses. If discussion runs long, shorten synthesis and current-project detail while preserving time to explain the independent research agenda.

## Build and verification

Use the build commands in `README.md`. The prepared PDF is `seminar.pdf`; the narration is `seminar-narrative.md`. Compact delivery cues remain embedded in the LaTeX source. Check changed slides visually and verify main-slide numbering, duration sums, note timings, page count, and the absence of incremental overlays before delivery.
