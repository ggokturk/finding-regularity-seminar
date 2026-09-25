# Faculty seminar editorial notes

The deck has **25 main slides**, followed by a divider and **14 backup slides**: **40 static PDF pages**. The title is **Finding Regularity: High-Performance Computing for Graph Algorithms**.

Prepared narration totals **30:50**. Reserve **35 minutes** for delivery with pauses and brief interruptions, with **40 minutes as the ceiling** in the one-hour seminar and interview session. This leaves roughly 20–25 minutes for discussion and interview questions. Timings are rehearsal targets, not measured delivery times.

## Opening revision for a CS faculty audience

Slide 9 combines the original **Component labels across samples** and **Reconstructing sampled edges** slides directly. The right column retains the 1--4--3 component, minimum-label propagation equation, and per-sample lane interpretation. The left column retains the stored seeds 5, 4, 14, 7; XOR with edge hash 6; results 3, 2, 8, 1; threshold 7; and the edge-membership equation.

The narration explains edge reconstruction on the left first, then uses those active edges in the component-label update on the right. It explicitly changes from the earlier directed running example to the undirected component algorithm. This preserves the original technical concepts for a CS faculty audience and connects directly to **Compute more, move less** on slide 10.

The merged slide retains the original two slides' combined **1:45** explanation time. The separate copies added to backup in the previous revision have been removed. The existing implementation backup retains the detailed hash definition. All later cumulative timings are adjusted by 30 seconds; the prepared talk now totals 30:50.

The shorter HPC introduction and early spoken graph-learning connection remain. The influence-maximization script traces one sample and then its Monte Carlo average. Slides, source comments, embedded cues, and the narrative timing table are synchronized. Every slide is static.

## Technical boundaries retained

- Main slide 9 explicitly treats undirected graphs. The minimum-label update identifies connected components; it is not a directed reachability algorithm.
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

- **Slide 8, ending at 8:10:** slowly trace the scattered column and adjacent row. Keep the distinction between logical state accesses and physical memory transactions.
- **Slide 9, ending at 9:55:** trace the XOR calculation on the left, then explain the component update on the right. Connect repeated propagation passes to the need for repeatable edge decisions.
- **Slide 10, ending at 11:10:** identify the stored graph write/read that fusion removes, then state the result and its sequential comparison scope.
- **Slide 12, ending at 14:05:** distinguish a register value from a count, then trace the inactive lane through the merge.
- **Slide 14, ending at 16:45:** follow one sample identity across assignments. Explain why skipping a warp differs from removing a sample's contribution.
- **Slides 17–18, ending at 22:45:** demonstrate locality opportunity, then explain conditional residual uniformity.
- **Slide 19, ending at 23:45:** state the historical measurement scope once.
- **Slide 24, ending at 30:20:** present the scheduling hypothesis as proposed work, with tuned baselines, unseen workloads, total cost, and fixed accuracy as evaluation criteria.

The 35-minute delivery slot includes about four minutes beyond the prepared narration. Rehearse aloud with pointing and pauses. If discussion runs long, shorten synthesis and current-project detail while preserving time to explain the independent research agenda.

## Build and verification

Use the build commands in `README.md`. The prepared PDF is `seminar.pdf`; the narration is `seminar-narrative.md`. Compact delivery cues remain embedded in the LaTeX source. Check changed slides visually and verify main-slide numbering, duration sums, note timings, page count, and the absence of incremental overlays before delivery.
