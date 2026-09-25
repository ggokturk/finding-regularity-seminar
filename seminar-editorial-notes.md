# Faculty seminar editorial notes

The deck has **26 main slides**, followed by a divider and **14 backup slides**: **41 static PDF pages**. The title is **Finding Regularity: High-Performance Computing for Graph Algorithms**.

Prepared narration totals **30:50**. Reserve **35 minutes** for delivery with pauses and brief interruptions, with **40 minutes as the ceiling** in the one-hour seminar and interview session. This leaves roughly 20–25 minutes for discussion and interview questions. Timings are rehearsal targets, not measured delivery times.

## Opening revision for a CS faculty audience

Slide 9 is devoted to **Hash-based edge sampling**. The large XOR threshold equation and the original four-sample calculation fill the slide. Its narration connects reconstruction, fused sample updates, and DiFuseR scheduling; component labels are introduced later.

Slide 11 uses generic state entries to motivate the remaining vertex-by-sample memory cost. Slide 12 puts **MixGreedy's component labels** beside **HyperFuseR's count-distinct sketch registers**. It retains the 1--4--3 example and minimum-label equation on the left and the original leading-zero table on the right. The comparison establishes what each representation stores and how it merges information. MixGreedy is credited for the component computation; the presenter's contribution is accelerating propagation through fused sampling and vectorization.

Slide 13 preserves the worked four-sample masked register merge on its own slide. This keeps the exact/approximate representation comparison readable and gives the active-edge mask, inactive sample, and idempotent maximum operation their own explanation.

The exact expression `X_r XOR h(u,v)` returns on the DiFuseR scheduling slide, now slide 15, and in the methodology summary on slide 21. FASST sorts input keys with their sample state; it does not sort each edge's XOR outputs.

Timing is redistributed: slide 9 takes 1:25; the representation comparison takes 1:20; masked sketch propagation takes 1:10. The full prepared delivery remains **30:50**. All slide numbers, cumulative timings, narration, and embedded cues are synchronized. Slides remain static.

## Technical boundaries retained

- The component-label example on slide 12 explicitly treats undirected graphs. Its minimum-label update identifies connected components; it is not a directed reachability algorithm.
- The main-slide XOR construction uses a symmetric hash for undirected edges. Reusing keys establishes repeatability across passes and directions, not joint edge independence.
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

- **Slide 8, ending at 8:10:** trace the scattered column and adjacent row; distinguish logical accesses from physical memory transactions.
- **Slide 9, ending at 9:35:** focus on the XOR sampling rule, its numeric example, and its roles in reconstruction, vectorization, and DiFuseR scheduling.
- **Slide 10, ending at 10:50:** explain the avoided stored-graph write/read, then state the result and its sequential comparison scope.
- **Slide 12, ending at 12:55:** introduce component labels and count-distinct registers together. Credit MixGreedy and distinguish exact component identity from a rare-pattern statistic.
- **Slide 13, ending at 14:05:** trace the inactive sample through the masked merge; explain idempotence and the accuracy/rebuilding scope.
- **Slide 15, ending at 16:45:** follow a sample identity across assignments; explain whole-warp skips without removing sample contributions.
- **Slides 18–19, ending at 22:45:** explain locality opportunity, then conditional residual uniformity.
- **Slide 20, ending at 23:45:** state the historical measurement scope once.
- **Slide 25, ending at 30:20:** present the future scheduler as proposed work with tuned baselines, unseen workloads, total cost, and fixed accuracy.

The 35-minute delivery slot includes about four minutes beyond the prepared narration. Rehearse aloud with pointing and pauses. If discussion runs long, shorten synthesis and current-project detail while preserving time to explain the independent research agenda.

## Build and verification

Use the build commands in `README.md`. The prepared PDF is `seminar.pdf`; the narration is `seminar-narrative.md`. Compact delivery cues remain embedded in the LaTeX source. Check changed slides visually and verify main-slide numbering, duration sums, note timings, page count, and the absence of incremental overlays before delivery.
