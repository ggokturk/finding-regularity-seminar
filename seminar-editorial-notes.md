# Faculty seminar editorial notes

The seminar has **25 main slides**, timed at **30:40**, followed by a divider and **14 backup slides**. The title is **Finding Regularity: High-Performance Computing for Graph Algorithms**. The user's simplified background and HPC introduction are retained. The complete spoken draft and cumulative timings are in `seminar-narrative.md`; compact cues are embedded in `seminar.tex`.

## What this revision changes

Slide 6 displays all content together. The gray outer block summarizes seed selection and queue updates. The expanded teal simulation loop details edge sampling, frontier traversal, visited state, termination, and distinct-vertex counting. The reachable-set definition appears inside traversal; the estimator follows the simulation loop. The right side contains the visual example.

The opening script allocates 20, 30, and 50 seconds to slides 1–3, giving slide 7 a full minute while retaining the 6:05 opening and 30:40 total. Slide 4 traces one active and one stopped lane. Slide 6 introduces the stochastic graph example before explaining the formulas and locating the repeated traversals inside the optimizer.

Slide 6 exposes a concrete lazy-greedy influence optimizer using nested blocks: source selection until the budget is reached, priority-queue candidate search until a current top score is found, and a simulation loop that estimates additional reachability. The spoken explanation states why cached gains remain bounds for a fixed sampled objective. The right side illustrates directed reachability from source 1 in two sampled graphs: reached sets {1,2,4} and {1,3,4}. Only active edges are shown. The optimizer is context; the spoken explanation focuses on traversal and averaging reachability counts. No figure attribution is displayed. This makes the repeated traversal cost visible inside the application.

The title's “Finding Regularity” theme is explicit in the opening, research question, representation transition, future agenda, and closing. Regularity means reusable data, nearby accesses, and similar work executed together. The story distinguishes structure found across samples from uniform updates introduced through compact representations, then reaches sampling-aware locality when exact alignment breaks down. Ongoing fusion and trajectory interleaving extend the broader bottleneck-driven methodology through reduced data movement and latency overlap. The closing recalls sample structure, compact representations, and sampling-aware schedules; the agenda asks when these choices can become systematic. The slide count and 30:40 delivery target are unchanged.

The exact research question is now the first spoken sentence. Repeated introductory explanations are shorter, while the first execution transformation receives more time. The background remains one compact slide, with the degrees directly beneath Sabancı University, research keywords, and consulting company names.

Slide 4 now explains its illustrations with concrete operations. Four labeled lanes load adjacent array elements, multiply by two, and store adjacent outputs. The graph illustration also has four labeled lanes: neighbor IDs route them to separated vertex-state locations through crossing arrows. New vertices have different amounts of expansion work, while already-seen vertices stop. Ellipses identify omitted memory locations. The two illustrations directly contrast adjacent data and equal work with scattered data and unequal work.

Slide 8 now shows two explicit lane-to-state mappings in vertex-major storage. Lanes assigned to different neighbors access different vertex rows; lanes assigned to samples for one edge access adjacent state. Shaded cells identify one vector's updates. The comparison shows four random reads and likely four random writes versus one sequential read and write, non-vector versus vector CPU compute, and GPU warp divergence versus a masked update with shared control flow. Contributing work can still vary across sample lanes. These are logical access patterns, not architecture-independent byte or transaction counts. The narrative retains active masks and batch-size tradeoffs.

Slide 17 uses the same eight illustrative seeds before and after sorting. Four-way groups request four distinct adjacency rows before sorting and two afterward at the next step. The spoken example then explains how the paired walks can diverge. This demonstrates the locality mechanism without claiming permanent alignment or presenting the toy example as a measured result.

The residual-range example remains in the main talk, with 90 seconds allocated to its invariant. Detailed fallback and exact-replay distinctions stay in backup. The SABA evidence qualification is stated once in the main narrative; reported measurements are unchanged.

The generic application list is replaced by active research: NeuralBloom, Battus, and trajectory execution. The final agenda proposes a specific first deliverable: an adaptive scheduler that chooses whether to preserve order, regroup batches, or interleave trajectories, accounting for its own overhead.

## Assessment against the hiring-talk goal

| Criterion | Current treatment |
|---|---|
| Research identity | The research question connects choices about work grouping, stored state, and hardware placement to reuse, memory footprint, and efficient execution. |
| Accessibility | The HPC introduction explains algorithmic choices; the architecture slide defines lanes and dependent accesses. |
| Technical depth | State layout and masks, recoverable randomness, mergeable sketches, sample-space decomposition, and conditional sampling each receive a concrete explanation. |
| Intellectual contribution | The narrative connects each design decision to a cost or semantic constraint. First person describes the research reasoning without inventing coauthor roles. |
| Coherent program | Shared topology leads to state compression, scheduling freedom, and then trajectory scheduling when alignment fails. |
| Departmental fit | NeuralBloom provides a current systems contribution to graph learning, while the main talk remains about algorithms and architecture. |
| Independent agenda | The runtime project has a hypothesis, observations, permissible actions, a deliverable, and an evaluation plan. |
| Scope and pacing | The prepared talk is 30:40. Current projects occupy the final research-program sequence rather than becoming three additional technical talks. |

## Current-project source and scope notes

**NeuralBloom:** `../neuralbloom/README.md` and `../neuralbloom/paper/main.tex` describe fused GPU construction of NeuralWalker's walk indices, edge indices, and structural encodings. The seminar's approximately 2× compute result is supplied by the presenter in this conversation and labeled preliminary. It is scoped to walk computation, not full-training wall time. The seminar does not make a publication-acceptance or venue-year claim for this ongoing extension.

**Battus:** `../battus/paper/sections/abstract.tex`, `introduction.tex`, and `method.tex` describe approximate mean-field forward–backward reconstruction from sparse diffusion snapshots, followed by deterministic decoding. The seminar uses the method and research question, not a new performance claim. Reconstruction quality and graph-causal feasibility remain separate obligations.

**Trajectory project:** `../randomwalk/walk.h`, `metal.h`, and `AGENTS.md` describe a deterministic coordinate-based traversal, CPU interleaving, and a Metal GPU implementation. This prototype is distinguished from independent Monte Carlo walks. Agreement between implementations does not by itself establish a stochastic path distribution or approximation guarantee.

Only these repositories' descriptions and implementations were read. Their source files and experiments were not changed, and no benchmark runs were launched.

## Delivery priorities

- **Slide 8, ending at 8:30:** point to the highlighted cells before explaining the loop organization. The memory accesses are the point.
- **Slide 17, ending at 21:05:** follow one unordered group and its sorted counterpart. Explain which adjacency data the next step needs.
- **Slide 18, ending at 22:35:** show why each chosen neighbor leaves a uniform residual. Keep the full proof for questions.
- **Slides 22–23, ending at 28:15:** describe the active projects as evidence that the methodology is already expanding. Avoid turning them into paper summaries.
- **Slide 24, ending at 30:10:** make the proposed adaptive scheduler the concrete future project the committee can remember.

If discussion consumes two minutes during delivery, shorten the general introduction and repeated synthesis, and omit some project implementation details. Preserve the lane-mapping example, the SABA locality example, and the first future deliverable.

## Build

```sh
latexmk -pdf -interaction=nonstopmode -halt-on-error seminar.tex
```

The presentation copy is `seminar.pdf`. The appendix is excluded from the main-slide progress denominator. The complete narrative is separate from the compact Beamer notes.

Slide 9 now demonstrates identity-based reconstruction with two execution orders. Edge a in sample 1 recovers 0.20 and remains active; edge b recovers 0.80 and remains inactive, with both probabilities 0.5. The rule sits between the two orders, and the takeaway connects reconstruction to avoiding stored sampled graphs. Illustrative values and a fixed generator seed are explicit. The spoken narrative distinguishes preservation of a realized graph from distributional correctness, retaining the separate statistical obligation.
