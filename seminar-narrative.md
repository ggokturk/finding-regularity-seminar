# Seminar speaker narrative

Prepared delivery: **30:40** across 25 main slides. This leaves about 9 minutes for questions in a 40-minute session. Rehearse aloud; the timings include explanation and pauses.

Use first person for your intellectual decisions and “we” for joint results. The SABA performance scope is stated once in the main narrative. Current projects are distinguished from completed work, and the preliminary NeuralBloom compute result is not a full-training speedup.

| Slide | Topic | Time | Cumulative |
|---|---|---:|---:|
| 1 | Opening | 0:20 | 0:20 |
| 2 | Research background | 0:30 | 0:50 |
| 3 | High-performance computing | 0:50 | 1:40 |
| 4 | The architectural mismatch | 1:30 | 3:10 |
| 5 | The research question | 0:35 | 3:45 |
| 6 | Influence maximization | 1:20 | 5:05 |
| 7 | The hidden regular dimension | 1:00 | 6:05 |
| 8 | Reorganizing execution | 2:25 | 8:30 |
| 9 | Reconstructing randomness | 1:15 | 9:45 |
| 10 | Compute more, move less | 1:15 | 11:00 |
| 11 | State becomes expensive | 0:45 | 11:45 |
| 12 | Compact mergeable state | 2:10 | 13:55 |
| 13 | CPU, GPU, and distributed mappings | 1:00 | 14:55 |
| 14 | Sample-space scheduling | 1:40 | 16:35 |
| 15 | Distributed evidence | 1:00 | 17:35 |
| 16 | Random walks break alignment | 1:15 | 18:50 |
| 17 | Bouquet arrangement | 2:15 | 21:05 |
| 18 | The corrected sampler | 1:30 | 22:35 |
| 19 | Cache behavior and evidence scope | 1:00 | 23:35 |
| 20 | A coherent methodology | 0:50 | 24:25 |
| 21 | Limits and cost models | 0:40 | 25:05 |
| 22 | Ongoing work in graph learning | 1:25 | 26:30 |
| 23 | Reconstruction and trajectory execution | 1:45 | 28:15 |
| 24 | An adaptive-scheduling project | 1:55 | 30:10 |
| 25 | Closing | 0:30 | 30:40 |

## 1. Opening — 0:20

How can we reorganize irregular computation so parallel hardware can execute it efficiently? I study this question through graph algorithms. I will show how shared structure across computations lets us change their execution order, representation, and hardware mapping.

## 2. Research background — 0:30

I completed my BSc, MSc, PhD, and PostDoc at Sabancı University. My work combines high-performance computing, parallel algorithms, and probabilistic graph computation. Alongside that research, I have done technical consulting for the organizations listed here. Today I will focus on how algorithms interact with real machines.

## 3. High-performance computing — 0:50

High-performance computing is about designing algorithms and systems to solve demanding problems efficiently.

There are several kinds of algorithmic decisions here. We can reduce redundant work, change how data is represented, reorganize the computation, or trade accuracy for cost when the application permits it.

Each decision interacts with the hardware: multicore CPUs, vector units, GPUs, and clusters. My research asks which decisions help when computation is irregular.

Sometimes reducing the expensive work means doing more cheap arithmetic to avoid moving data. We will see that trade repeatedly. First, let me show why simply adding parallelism can fail.

## 4. The architectural mismatch — 1:30

A lane is one position in a group executing an operation in parallel. On the left, four lanes load adjacent array elements, multiply each by two, and store adjacent outputs. Addresses are predictable and the work is equal.

On the right, the lanes read neighbor IDs. Those IDs determine which vertex-state locations must be accessed. Follow lane zero's arrow: it reads vertex seven's state, discovers a new vertex, and has two edges to process.

Lane one reaches vertex eighty-three, which has already been seen, so it stops. Another lane has five edges to process. The arrows lead to separated memory locations, and the lanes have different amounts of useful work.

That creates two costs: waiting for data and leaving lanes idle. More lanes do not automatically remove either cost. To get useful performance, we need to consider both the memory accesses and which operations execute together.

## 5. The research question — 0:35

By regularity, I mean predictable memory access and similar work across parallel lanes. I look for structure that lets us reuse data, access nearby memory locations, and give lanes similar operations. That structure can be partial or temporary. The choices are which work runs together, what state is stored, and where it executes. Each change must preserve the required result, sampling distribution, or accuracy target.

## 6. Influence maximization — 1:20

Influence maximization asks which k seeds have the greatest combined influence. Their reach can overlap, so a seed's value depends on those already selected.

Start with the graph on the right. Each edge succeeds independently with a given probability. A simulation keeps the successful edges.

From source one, the first sample reaches two and four. The second reaches four and then three. Including the source, both reach three vertices. Sample three reaches only one and three; its edge from two to four is disconnected from the source. Sample four reaches all four vertices. We will follow these same four samples onto the next slide.

The teal loop evaluates a trial seed set S. Its first formula defines the vertices reachable from any seed in S. Sample the edges, follow active edges to unseen vertices, and stop when the frontier is empty. Visited state prevents double-counting. The formula at the bottom averages the reached-set sizes. The optimizer compares these estimates to determine the additional reach from adding a seed.

We therefore revisit the same topology across both candidates and simulations. That repeated access is the cost I want to reduce.

## 7. The hidden regular dimension — 1:00

The bunkbed picture led me to think of simulations as stacked graph copies. I expected many samples to use overlapping edges. Could processing them together turn that overlap into dense groups of useful updates?

Follow edge one to four across the layers. It is active in samples one, two, and four, but absent from sample three. The membership set on the right records those same decisions on one graph.

Now hold that edge fixed and assign the four samples to four lanes. One topology access can serve several updates. Each sample retains its own state, and the inactive sample is masked.

Shared topology gives us reuse. How well the active work lines up becomes a scheduling question later. First, the next slide shows what this mapping does to memory access.

## 8. Reorganizing execution — 2:25

The highlighted cells show one group of four lanes. Both diagrams use vertex-major storage: one row contains the states of several samples for one vertex. The full computation must eventually cover the same edge–sample work; we are changing which updates execute together.

On the left, all four lanes work on sample zero, but each follows a different neighbor. Those neighbor IDs become four different vertex rows. Even if the adjacency list itself is contiguous, the state accesses are separated in memory. With many samples, the distance between rows becomes substantial.

On the right, I hold the edge to vertex seven fixed. The lanes process four samples, so their destination states sit next to each other in the same row. One edge access serves several decisions, and a vector load can fetch adjacent state. That is the architectural change: shared topology and contiguous sample state.

For the highlighted work, the left needs four random reads and likely four random writes. The right groups the state access into one sequential read and one sequential write. These describe the access pattern; the number of hardware transactions depends on the architecture. On CPUs, the reorganized version uses vector instructions instead of non-vector compute. On GPUs, different neighbor paths can cause warp or workgroup divergence and stalls on the left. Independent thread scheduling helps with divergent execution, but it does not make the scattered memory accesses contiguous; that cost remains high. The masked update on the right shares control flow, avoiding branch divergence within that update. But the contributing work still varies: an inactive sample lane does not produce a useful update.

There is an algorithmic constraint. Samples can have different frontiers and different active edges. The implementation retains that state and masks inactive lanes. Sharing an edge does not mean forcing every sample to take it.

Larger batches can improve reuse but increase state footprint and inactive work. The right batch size depends on the memory hierarchy and the workload. This is why the contribution is an algorithm-and-layout transformation, with vector instructions implementing that choice. But changing execution order raises another question: how do we recover the same edge decisions in each sample?

## 9. Reconstructing randomness — 1:15

We changed the execution order. How do we preserve the same sampled graphs?

An ordinary random-number stream associates successive draws with execution order. If I change which edge comes first, I can change the realized sampled graph—even if the overall sampling distribution remains valid.

I therefore made each decision recoverable from the edge and sample identity, using a fixed generator seed. In this example, both edge probabilities are one half. Edge a in sample one recovers point two, so it is active. Edge b recovers point eight, so it is inactive. Reverse their execution order: each identity still recovers its original value and decision.

The threshold equation is the membership test. The important contribution is making execution order a systems design choice while retaining each sample's decisions.

Repeatability is only one obligation: the pseudorandom mapping must also support the required sampling distribution. A deterministic hash alone does not prove independence.

We can now reconstruct edge membership when needed, without storing each sampled graph. That sets up the next tradeoff: extra computation to avoid data movement.

## 10. Compute more, move less — 1:15

We deliberately add hashing, comparisons, and masking. That can still be faster because those operations replace repeated accesses and stored intermediate graphs. Instruction count is useful only in the context of the bottleneck.

The first paper reports roughly three to twenty-one times improvement from fused sampling in the sequential comparison against MixGreedy, on the three graphs where that reference completed. I use this comparison because it makes the contribution of fusion more visible than the much larger complete-system speedups.

This particular variant generates decisions on the fly but still processes samples separately. It isolates the value of avoiding materialized sampled graphs; it does not isolate the sample-parallel transpose. Batching and memoization are further changes in the full method.

The broader principle is compute more, move less. Once topology traffic is reduced, however, another cost becomes visible: the amount of state maintained across samples.

## 11. State becomes expensive — 0:45

So far, we found regularity across samples. Next, we change the representation so that the state and its updates become more uniform.

Sharing topology still leaves state proportional to vertices times samples, even with one label per pair. What does the query actually need? Influence estimation needs the number of reachable vertices, and may not need their identities. A compact summary can retain that information approximately. This introduces an accuracy tradeoff: it is an algorithmic choice as well as a memory optimization.

## 12. Compact mergeable state — 2:10

HyperFuseR uses a compact count-distinct summary. The intuition is that a larger set is more likely to contain a hash value with a rare bit pattern, such as many leading zeros. A small register retains the strongest such observation. Several sample-aligned registers provide the information used in the estimate.

The crucial operation is merge. For corresponding registers, we take the maximum. This is fixed-width arithmetic that maps well to vectors. More importantly, it is idempotent: merging the same information twice changes nothing. If two graph paths reach the same vertex, the summary does not treat those paths as two distinct vertices.

In this algorithm, a register position is associated with a stochastic sample, and propagation is gated by that sample's edge decisions. We should not confuse the array with an arbitrary exact reached set. The representation estimates the quantity needed by the influence query.

There is a price: sketch error and decisions about rebuilding the summaries as the selected source set changes. The method includes error-adaptive rebuilding; compression is not free accuracy.

The architectural result is compact, contiguous state with a uniform, cheap merge. Here we introduce useful regularity through the representation we choose. That choice must serve the query and its accuracy requirements.

## 13. CPU, GPU, and distributed mappings — 1:00

The sample dimension survives across architectures. CPU vector lanes update adjacent sample states. On a GPU, threads in a warp work on samples for one vertex or edge, making their state accesses nearby. A warp is a group of threads executing together.

Across GPUs, I partition samples. Each device propagates its own sample states and retains the edges those samples require. Edges may overlap between devices; this is not a disjoint vertex partition. Global influence decisions still require reductions of local estimates.

The abstraction is portable, while batch sizes, scheduling, and communication remain architecture-specific. Sample assignment then creates another opportunity: choosing which samples should run together.

## 14. Sample-space scheduling — 1:40

Mapping samples to lanes exposes another degree of freedom: which samples should be neighbors? In the left-hand schematic, each warp contains a mixture of samples where this edge is active and inactive. Both warps must execute the update, with some threads idle. On the right, active decisions are concentrated, and one entire warp can skip the work.

FASST sorts the existing sample keys before assigning them to warps and devices. With the fused decision construction, related keys can produce related decisions for a fixed edge. It is a way to improve grouping across many edges, not an oracle that perfectly sorts the active mask separately for every edge.

The same grouping matters at device scale. If no sample assigned to a device includes an edge, the device can omit that edge from its local graph. This can reduce both local storage and redundant topology processing.

The estimator averages over the same sample identities. Their order is irrelevant to that aggregate, so schedule freedom becomes a resource. The benefit depends on the probabilities and the available samples: if nearly every device needs nearly every edge, the opportunity is much smaller.

## 15. Distributed evidence — 1:00

Here is the distributed evidence I want to emphasize. With a fixed workload, DiFuseR reaches a geometric-mean speedup of 5.64 on eight A100 GPUs relative to one GPU. The multi-GPU experiments use two GPUs per node. The single-GPU fit requirement excludes Friendster from this comparison.

This is useful scaling, not perfect scaling. Local graph overlap, work balance, and global reductions all affect the result. The improvement above two at the two-GPU point also reminds us that partitioning can change the amount of local graph work and the memory footprint.

The architectural lesson is that stochastic samples are a useful unit of distributed decomposition. That completes the aligned-sample story. The harder question is what remains when the samples cannot stay aligned.

## 16. Random walks break alignment — 1:15

A random walk takes us into a harder regime. The next vertex depends on a random choice at the current vertex, and the next adjacency address depends on that chosen vertex. Independent walks quickly move to different regions of the graph.

In the earlier algorithms, I could hold a graph edge fixed and process many samples against it. Here, insisting that every lane use the same edge is no longer natural. Each trajectory has a sequential dependency chain, even though many trajectories can run independently.

Assigning one walk to each vector lane exposes parallel work but may also expose many unrelated cache misses at once. The problem has changed. We now need to schedule trajectories so that they are likely to have useful locality. This is where the methodology must generalize beyond loop transposition.

## 17. Bouquet arrangement — 2:15

Here is what sorting actually changes. Every walk starts at the same source, which has four neighbors. For this illustration, the first choice partitions the normalized seed range into four equal intervals. The same eight independently drawn seeds appear on both sides.

In the first unordered group, the seeds choose D, A, C, and B. At the next step, the lanes need four different adjacency rows. The second group also needs four. Sorting produces A, A, B, B in the first group and C, C, D, D in the second. Each group now needs adjacency data for only two distinct current vertices.

That is the locality opportunity. Lanes can reuse row pointers and adjacency data, and consecutive groups can also retain useful data in cache. This example does not imply a particular measured speedup or number of cache misses; it exposes the mechanism we want to encourage.

Exact alignment need not persist. The two seeds that reached A retain different residual positions. If A has neighbors E and F, they can take different next steps. As trajectories evolve, degrees, interval boundaries, and fallback sampling can separate them further.

The method therefore uses statistical structure to improve a schedule, rather than requiring every lane to follow the same path. It is also specific to the sampling construction: sorting arbitrary seeds from an arbitrary generator need not produce this behavior.

The remaining question is whether the same construction can give valid multi-step walks. That requires consuming the random information used by each choice, while retaining an appropriate residual for the next one.

## 18. The corrected sampler — 1:30

The requirement is conditional: given the path so far, the next choice must be uniform over the current neighbors. Reusing random information already revealed by earlier choices can break that requirement.

In this small integer example, x is uniform over twelve values and the current vertex has three neighbors. Divide the range into three equal intervals of size four. The interval selects the neighbor; the position within it becomes the residual.

Each neighbor has four possible inputs. After choosing any neighbor, the residual is still uniform over zero through three. Each neighbor–residual pair has exactly one preimage. That invariant supports the next step and preserves useful grouping in the early levels.

When the residual is too small or falls in a rejected remainder, the walk permanently switches to fresh unbiased conventional sampling. The details are in backup.

This is the sampler-validity argument. Separately, permuting a fixed set of valid walks leaves their average unchanged in exact arithmetic. A correct sampler creates the work; scheduling chooses how to execute it.

## 19. Cache behavior and evidence scope — 1:00

The reported measurements illustrate why we must inspect memory behavior. Direct AVX2 vectorization produced more last-level cache misses than scalar walks. The historical SABA implementation reported a large reduction in misses and a 3.38 times walk-processing speedup over its HASH baseline at this walk count.

These tables describe the earlier sampler and do not validate the corrected residual-range version. That qualification remains explicit on the slide.

The systems lesson is that vectorization and locality are distinct. Exposing more simultaneous work can increase pressure on memory. The scheduling question is how to make that work share data, and the benefit must ultimately be evaluated at the required accuracy and total execution cost.

## 20. A coherent methodology — 0:50

We found regularity in shared topology across samples, introduced uniform operations through compact representations, and used sample ordering to improve coherence. When trajectories diverged, sampling structure still offered opportunities for locality.

The contribution is the connection between an algorithmic choice and the architectural cost it changes. It is not a universal instruction to sort or compress everything. We need a bottleneck, a legal transformation, and a reason that transformation should reduce total cost. That perspective now guides several ongoing projects.

## 21. Limits and cost models — 0:40

These transformations pay off only when there is enough independent work and enough reusable structure. Ordering can lose on small batches or trajectories that quickly spread apart. Sketches are inappropriate when exact identities are required.

A wider batch can improve reuse while increasing state footprint. A GPU can expose more parallelism while adding transfer costs. Those competing costs motivate an adaptive policy, rather than one fixed schedule for every workload.

## 22. Ongoing work in graph learning — 1:25

The broader method is to identify the expensive work and change how we execute it. NeuralBloom applies that approach to NeuralWalker's input construction: walks, edge identities, and structural encodings. The neural model stays unchanged.

A direct tensor implementation can dispatch work at successive walk positions and move intermediate state through global memory. The fused approach lets one CUDA thread advance a walk while retaining private state, and constructs the required encodings in the same kernel. Independent walks provide the parallel dimension.

My current experiments show roughly a twofold compute speedup. I am presenting that as a preliminary result for the walk computation, not as a twofold improvement in complete model training.

The main opportunity here is reducing data movement and dispatch overhead through fusion. The next questions concern batch size, fusion depth, and when the benefit is outweighed by register pressure or insufficient parallel work.

## 23. Reconstruction and trajectory execution — 1:45

Two other ongoing projects explore different algorithmic freedoms. Battus reconstructs missing diffusion states between sparse graph snapshots. It replaces a more elaborate proposal-and-sampling pipeline with approximate per-vertex probabilities, forward–backward inference, and decoding. These operations become regular graph passes that can map to CUDA.

The research question is which inference stages and intermediate state are actually needed for useful reconstruction. That is an approximation question, so reconstruction quality and causal consistency need their own evaluation; this is not merely a reordered exact algorithm.

The trajectory project explores path coordinates and execution schedules. Its current prototype uses deterministic starting coordinates, updates those coordinates along the graph, and interleaves several trajectories on the CPU. It also maps trajectories to GPU threads through Metal.

When locality is limited, interleaving independent trajectories may still let the processor overlap memory latency. This is another response to the architectural bottleneck. Deterministic trajectory rules also need their own approximation analysis; they are not interchangeable with independent Monte Carlo walks.

Together these projects extend the program across execution order, representation, and approximation. They also identify which parts a future runtime could choose automatically and which require an explicit contract from the application.

## 24. An adaptive-scheduling project — 1:55

Can we systematically identify useful regularity, and decide when exploiting it is worth the cost? My first project is an adaptive scheduler for irregular trajectories. The hypothesis is that short observations of repeated vertices, active lanes, and memory stalls can predict whether regrouping will save more time than it costs.

The policy would choose between leaving work in its current order, regrouping a batch for reuse, and interleaving trajectories to overlap latency. The first deliverable is a working policy evaluated against tuned static schedules on held-out graphs and sampling workloads. All timings must include observation and scheduling overhead.

The application supplies the legal freedom: which tasks can be reordered, how random streams belong to tasks, and which accuracy target must remain fixed. A deterministic trajectory approximation and an independent Monte Carlo sampler cannot silently share the same contract.

Graph walks and NeuralBloom are concrete initial workloads. Diffusion inference then tests whether the approach extends to different state and dependencies. CPU–GPU placement and representation selection are later steps, once the scheduling decision is understood.

This makes the agenda independent of any one graph problem. The central research contribution would be a systematic connection between semantic constraints, inexpensive online observations, and decisions that improve performance on real hardware.

## 25. Closing — 0:30

Finding regularity took three forms: sample structure that lets us share topology, compact representations with uniform updates, and sampling-aware schedules that encourage reuse as trajectories diverge. My goal is to make these choices systematic, preserving correctness and accounting for their total cost. Thank you; I look forward to your questions.
