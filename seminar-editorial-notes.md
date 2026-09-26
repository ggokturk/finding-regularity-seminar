# Faculty seminar editorial notes

The deck has **25 main slides**, followed by a divider and **14 backup slides**: **40 static PDF pages**. The title is **Finding Regularity: High-Performance Computing for Graph Algorithms**.

Prepared narration totals **30:50**. Reserve **35 minutes** for delivery with pauses and brief interruptions, with **40 minutes as the ceiling** in the one-hour seminar and interview session. This leaves roughly 20–25 minutes for discussion and interview questions. Timings are rehearsal targets, not measured delivery times.

## Opening revision for a CS faculty audience

Slide 7 combines the shared graph's sample-membership labels with the hash-based sampling rule and its numeric example. The highlighted edge 1 to 4 is labeled with samples {1,2,4}; the adjacent XOR calculation reconstructs precisely that membership. The standalone sampling slide is folded into this explanation. Directed arcs retain their identities; symmetric hashing is stated only for the undirected case.

Slide 8 holds the sample batch fixed and compares two actual storage layouts: `state[r][v]` versus `state[v][r]`. The batch consists of vertex 4's state in four samples. Its entries are |V| positions apart in sample-major storage and adjacent in vertex-major storage. Both matrices contain the same state. This replaces the previous comparison of two execution groupings that both used vertex-major storage.

The distinction is now explicit: slide 7 explains graph membership and reconstruction; slide 8 explains state placement. Hash-based sampling remains central and its scheduling benefit is previewed without introducing lock-free execution or update idempotence here.

Slide 10 introduces generic per-sample state costs. Slide 11 compares MixGreedy's component labels with HyperFuseR's count-distinct sketches. Slide 12 retains the masked register merge. The XOR construction returns in FASST on slide 14 and the methodology summary on slide 20.

Slide 7 receives the combined 2:25 previously allocated to shared topology and hash-based sampling. Slide 8 retains 2:25 for the memory-layout comparison. The prepared total remains **30:50**, with 25 main slides. Narration, cues, and numbering are synchronized; every slide is static.

## Technical boundaries retained

- The component-label example on slide 11 explicitly treats undirected graphs. Its minimum-label update identifies connected components; it is not a directed reachability algorithm.
- The main-slide XOR construction distinguishes ordered directed edges from symmetric undirected hashes. Reusing keys establishes repeatability across passes and directions, not joint edge independence.
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

- **Slide 7, ending at 7:10:** trace edge 1 to 4 from the graph into the XOR calculation and back to its membership {1,2,4}. Explain that membership sets are reconstructed, not stored.
- **Slide 8, ending at 9:35:** keep the batch fixed while comparing actual storage layouts. Identify which dimension is contiguous and the stride between lane accesses.
- **Slide 9, ending at 10:50:** explain the avoided sampled-graph write/read and state the sequential comparison scope.
- **Slide 11, ending at 12:55:** introduce component labels and sketches together; credit MixGreedy and distinguish component identity from a rare-pattern statistic.
- **Slide 12, ending at 14:05:** trace the inactive sample through the masked merge and explain accuracy/rebuilding scope.
- **Slide 14, ending at 16:45:** follow sample identities and connect scheduling to the XOR construction on slide 7.
- **Slides 17–18, ending at 22:45:** explain locality opportunity, then conditional residual uniformity.
- **Slide 19, ending at 23:45:** state the historical measurement scope once.
- **Slide 24, ending at 30:20:** present the future scheduler as proposed work, with tuned baselines, unseen workloads, total cost, and fixed accuracy.

The 35-minute delivery slot includes about four minutes beyond the prepared narration. Rehearse aloud with pointing and pauses. If discussion runs long, shorten synthesis and current-project detail while preserving time to explain the independent research agenda.

## Build and verification

Use the build commands in `README.md`. The prepared PDF is `seminar.pdf`; the narration is `seminar-narrative.md`. Compact delivery cues remain embedded in the LaTeX source. Check changed slides visually and verify main-slide numbering, duration sums, note timings, page count, and the absence of incremental overlays before delivery.
