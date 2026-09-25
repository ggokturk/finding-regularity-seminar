# Seminar speaker narrative

Prepared delivery: **31:10** across 26 main slides. This leaves almost 9 minutes for questions in a 40-minute session. Rehearse aloud; the timings include explanation and pauses.

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
| 9 | Component labels across samples | 0:50 | 9:20 |
| 10 | Reconstructing sampled edges | 0:55 | 10:15 |
| 11 | Compute more, move less | 1:15 | 11:30 |
| 12 | State becomes expensive | 0:45 | 12:15 |
| 13 | Compact mergeable state | 2:10 | 14:25 |
| 14 | CPU, GPU, and distributed mappings | 1:00 | 15:25 |
| 15 | Sample-space scheduling | 1:40 | 17:05 |
| 16 | Distributed evidence | 1:00 | 18:05 |
| 17 | Random walks break alignment | 1:15 | 19:20 |
| 18 | Bouquet arrangement | 2:15 | 21:35 |
| 19 | The corrected sampler | 1:30 | 23:05 |
| 20 | Cache behavior and evidence scope | 1:00 | 24:05 |
| 21 | A coherent methodology | 0:50 | 24:55 |
| 22 | Limits and cost models | 0:40 | 25:35 |
| 23 | Ongoing work in graph learning | 1:25 | 27:00 |
| 24 | Reconstruction and trajectory execution | 1:45 | 28:45 |
| 25 | An adaptive-scheduling project | 1:55 | 30:40 |
| 26 | Closing | 0:30 | 31:10 |

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

Start with the graph on the right and seed one. In the model, each edge succeeds independently with its given probability. A simulation keeps the successful edges, and we follow them from the seed.

The first sample reaches one, two, and four. The second reaches one, four, and three. Both counts are three. Sample three reaches only one and three: its edge from two to four is disconnected from the seed. Sample four reaches all four vertices.

Now average the counts: three plus three plus two plus four, divided by four, gives three vertices. That is our estimate from these four samples. The seed itself counts every time.

For a set of seeds, we count their combined reach, counting each vertex once. We compare the extra reach from adding different candidates. This repeats graph access across candidates and simulations. That repeated access is the cost I want to reduce.

## 7. The hidden regular dimension — 1:00

My design decision is to organize the computation across samples so one edge access serves several updates.

The stacked graph copies make that opportunity visible. Follow edge one to four across the layers. It is active in samples one, two, and four, but absent from sample three. The membership set on the right records those same decisions on one graph.

Hold that edge fixed and assign the four samples to four lanes. The topology access is shared. Each sample retains its own state, and the inactive sample is masked.

Shared topology gives us reuse. How well the active work lines up becomes a scheduling question later. First, the next slide shows what this mapping does to memory access.

## 8. Reorganizing execution — 2:25

The highlighted cells show one group of four lanes. Both diagrams use vertex-major storage: one row contains the states of several samples for one vertex.

On the left, all four lanes work on sample zero, but each follows a different neighbor. Trace the highlighted column: those neighbor IDs lead to four different vertex rows. Even if the adjacency list is contiguous, these state accesses are scattered.

On the right, I hold the edge to vertex seven fixed. Trace the highlighted row: the lanes process four samples, so their destination states sit next to each other. One edge access serves several decisions.

The comparison is four scattered state accesses versus one contiguous group of four states. It describes the layout of the data requested together. The number of hardware transactions depends on the machine. Adjacent state supports vector loads on CPUs and adjacent lane accesses on GPUs.

We still have to perform the required edge–sample work. The change is which updates execute together. Samples can have different active edges and frontiers, so each sample retains its own state and inactive updates are masked. An inactive lane contributes no useful update.

Larger batches can reuse more topology, but they also increase the state footprint and may include more inactive work. The useful batch size depends on the workload and memory hierarchy.

We have changed how work is grouped. Let me now make the update performed by each sample lane concrete.

## 9. Component labels across samples — 0:50

The earlier example used directed edges. Now consider undirected graphs, where a source reaches every vertex in its connected component.

Initialize each vertex label with its ID. Repeatedly take the smallest label among that vertex and its active neighbors. Here, vertices one, four, and three converge to label one.

This is the update for the lanes on the previous slide. Each lane maintains a label for a different sample, and inactive edges are masked. Label propagation is standard; my contribution is organizing it across samples.

Every pass must use the same sampled edges. How do we recover those decisions without storing every sampled graph?

## 10. Reconstructing sampled edges — 0:55

Store one random key per sample and a symmetric hash per edge. The random key identifies a sample; it is distinct from the influence source.

For edge one–four, XOR hash six with the four keys. The results are three, two, eight, and one. Threshold seven activates samples one, two, and four, reproducing the earlier membership pattern.

Reusing these keys recovers the same decision on every pass and in both directions. That establishes repeatability. Independent edge sampling requires separate justification.

I fuse this reconstruction with the propagation across samples. Labels remain in memory, but the sampled graphs need not be stored. We recover edge decisions when needed. What does that tradeoff save?

## 11. Compute more, move less — 1:15

Recovering an edge decision means we do not have to write that decision into a separate sampled graph. The upper pipeline samples the edges, stores the resulting graph, and reads it again during traversal. The lower pipeline has three steps: read the original topology, reconstruct the edge decision, and traverse immediately. These happen within the same traversal, without storing an intermediate sampled graph.

That adds arithmetic: a key operation, a comparison, and a conditional update. But it removes the stored intermediate graph and the traffic needed to produce and consume it. This is why instruction count alone is a poor predictor of performance here.

The sequential fused-sampling comparison reports approximately three to twenty-one times improvement over MixGreedy on the three graphs where that reference completed. Those samples still execute separately. This result supports generating decisions on demand; it does not isolate the sample-lane transformation from slide eight.

We can then combine reconstruction with sample batching. But sharing topology does not make all the state disappear. That is the next bottleneck.

## 12. State becomes expensive — 0:45

This is the same kind of matrix we saw earlier. Every vertex still has separate state for every sample. Even one exact label per pair gives us vertices times samples entries, and each traversal reads and updates them.

Return to sample one: the reached vertices were one, two, and four. Their contribution to the influence estimate is the count, three. That suggests a different question: can smaller state support the quantity we actually need?

The next method estimates reach with compact registers. The matrix remains, but its entries and update operations change. That is an approximation decision as well as an architectural one.

## 13. Compact mergeable state — 2:10

HyperFuseR uses count-distinct sketches. Let me first explain what one register stores. Hash each reached vertex, and count the leading zeros. In this toy example, the three vertices produce zero, two, and three leading zeros. The register keeps the maximum, three.

Three is not the count of reached vertices. It describes the rarest pattern seen. Larger sets are more likely to contain a hash with a longer zero prefix. A collection of these observations supports a cardinality estimate.

The useful property for graph propagation is how we combine summaries. On the right, each column belongs to a different stochastic sample. We have the current vertex's registers, a neighbor's registers, and the edge's active mask. In sample one, the maximum of two and four is four. Sample two is inactive, so its register stays one even though the neighbor has three. Samples three and four use the same maximum operation.

Now consider two paths reaching the same vertex. Its hash is unchanged, so processing that vertex again cannot increase the register just by repeating it. Maximum is idempotent: repeated information changes nothing.

This gives us smaller entries, contiguous sample state, and one uniform merge operation for active lanes. My representation choice serves both the query and the hardware. There is an accuracy cost: the method monitors discrepancies against Monte Carlo evaluations and rebuilds summaries when needed. The sketch is a fast influence oracle within that larger algorithm, not a replacement for every exact operation.

## 14. CPU, GPU, and distributed mappings — 1:00

Follow the same sample entries down the slide. On a CPU, a vector instruction updates adjacent sample states. On a GPU, neighboring threads process neighboring sample states for the same edge. The eight entries are schematic; the physical widths vary.

For multiple GPUs, I divide the sample dimension between devices. Each device keeps its own sample registers and the graph edges those samples require. An edge can be present on several devices, so this is not a partition into disjoint graph regions.

Sketch propagation stays local. Global source selection requires combining sketch statistics and communicating the selected source. The graph algorithm therefore gives us a useful decomposition, but it does not remove communication.

There is one more choice inside this mapping: which sample identities should occupy adjacent lanes and devices?

## 15. Sample-space scheduling — 1:40

For this edge, samples two, four, six, and eight are active. Look at the original assignment: both warps contain two active samples and two inactive samples. Both warps enter the update, although only half their lanes contribute.

Now follow the sample identities to the right. There are still eight samples and four active updates. But one warp contains the active samples, and the other can skip the edge's update altogether. We changed the assignment, not the sampled decisions.

FASST obtains this opportunity by sorting the existing random keys before assigning samples to warps and devices. The sample state must move with its key. This is one ordering used across the graph, not a separate perfect partition computed for every edge.

The fused decision construction matters: related keys can give related decisions for a fixed edge. Sorting arbitrary outputs from an arbitrary hash would not promise the same structure.

At device scale, the consequence is similar. If no local sample includes an edge, the device can omit that edge. Sample order therefore affects both lane utilization and the graph data each device needs. The aggregate estimate uses the same sample identities, so their placement becomes a scheduling choice.

## 16. Distributed evidence — 1:00

The reported fixed-workload comparison reaches a geometric-mean speedup of 5.64 on eight A100 GPUs relative to one GPU. The experiment uses two GPUs per node; Friendster is excluded because the one-GPU case does not fit.

The diagram explains the decomposition behind that result. Each device propagates sketches through its local samples and required edges. At source selection, devices combine sketch statistics and communicate the selected source.

Edge overlap, unequal work, and global reductions all limit scaling. Partitioning can also change the amount of local graph work and its memory footprint, so the superlinear two-GPU point should not be read as an arithmetic-throughput effect alone.

This completes the part of the story where we can keep an edge fixed across samples. Random walks will remove that convenient alignment.

## 17. Random walks break alignment — 1:15

My random-walk work estimates the importance of graph edges to connectivity. It requires many trajectories through the graph, so the cost of those walks becomes central.

Within one walk, I must read the current adjacency list, choose a neighbor, and then read that neighbor's list. I cannot know the next address before making the preceding choice.

Read the right-hand example one row at a time. All four walks start at the same vertex and can reuse one adjacency row. After one step, they occupy three vertices. After another step, they occupy four. These are distinct requested rows, not measured cache misses.

The walks can execute independently, but they no longer request the same data. Simply putting one walk in each lane exposes parallel work and potentially several unrelated memory stalls.

So the question changes: can I arrange trajectories so that nearby lanes are likely to request reusable adjacency data?

## 18. Bouquet arrangement — 2:15

This is the central scheduling idea in SABA. A bouquet is a group of walks processed together. We draw the random keys first and then choose their execution groups.

All eight walks in this example start at the same source. It has four equally likely neighbors. The first quarter of the key range chooses A, the second chooses B, and so on. These normalized keys are only an illustration of the range construction.

Trace the first unordered group: point eight three chooses D, point one four chooses A, point six three chooses C, and point two nine chooses B. At the next step, those lanes request four different adjacency rows. The second group also requests four.

Sorting the same keys gives A, A, B, B in the first bouquet and C, C, D, D in the second. Each group now requests only two distinct rows. That creates opportunities to reuse row pointers, neighbor data, and cache contents. It does not guarantee a particular number of physical memory transactions.

Crucially, the walks still own separate states. The two walks at A retain different positions inside the interval that selected A. They may choose different neighbors at the following step. Degrees, interval boundaries, and later fallback sampling can all break the grouping.

I am therefore using structure in the sampling process to improve the schedule, without requiring permanent agreement between trajectories. But that raises a deeper question: how do we use an initial random key for successive choices without reusing randomness that the earlier choices have already revealed?

## 19. The corrected sampler — 1:30

The next neighbor must be uniform conditional on the path so far. Reproducing a decision is not enough to establish that property.

Here, the current residual has twelve equally likely values and the vertex has three neighbors. Divide the range into three equal intervals. Each interval contains four inputs, so each neighbor is equally likely.

Take the highlighted input, seven. It belongs to interval one, so we choose the middle neighbor, index one. Its position inside that interval is three. We retain that position, with a new range of size four, for the next step.

Now look across the residual row. Whichever neighbor we chose, the remaining values are zero, one, two, and three, each equally likely. The choice revealed which interval we were in, but it did not reveal the position inside that interval. That is the invariant that allows the next step.

If the available range is exhausted or the input falls outside the equal-sized buckets, the walk switches permanently to fresh unbiased sampling. The full rule is in backup.

This establishes sampler validity under the independent-input assumptions. Reordering valid walk contributions is a separate argument. Sorting alone cannot repair an incorrect path distribution.

## 20. Cache behavior and evidence scope — 1:00

The orange bar returns to the opening systems question. Direct AVX2 vectorization produced 8.83 percent more last-level cache misses than scalar execution. More arithmetic lanes did not solve the memory problem.

The HASH baseline reduced the normalized miss count to 61.92; the reported SABA implementation reduced it to 3.55. The corresponding walk-processing comparison reports a 3.38 times speedup over HASH.

These are measurements of the earlier implementation, not runtime measurements of the corrected residual sampler I just explained.

The important connection is between the schedule and the addresses requested together. That is what the bouquet example exposed. When assessing an optimization, I want to see both the runtime result and evidence that the architectural bottleneck changed in the expected way.

## 21. A coherent methodology — 0:50

Across these projects, I have repeatedly changed the algorithm around a particular machine cost.

Repeated graph reads led to sample batching and reconstructed decisions. Large exact state led to compact registers with uniform merges. Inactive lanes led to sample ordering and device partitions. Dependent walk addresses led to grouping trajectories through their sampling structure.

The useful freedom was different each time: the order of execution, the representation, or the way random choices were constructed. The research task was to find that freedom, connect it to a hardware cost, and preserve the result or accuracy the application required.

The next question is when each transformation is actually worth its cost.

## 22. Limits and cost models — 0:40

Larger batches reuse more topology, but they also increase the state footprint and can include more inactive lanes. Ordering walks can improve early locality, but sorting costs time and the paths may soon separate.

The same tradeoff appears in sketching and processor placement: smaller state introduces estimation and rebuilding costs; GPU execution needs enough work to repay transfers.

So I measure total execution time at the required accuracy. These competing costs are also why I want an adaptive decision rather than a fixed setting for every graph and workload.

## 23. Ongoing work in graph learning — 1:25

NeuralBloom applies this reasoning to NeuralWalker's input construction. The neural model consumes walks and structural encodings. My work focuses on how we produce that representation efficiently.

With separate GPU operations, a walk step can write intermediate state that later operations read again. There is also dispatch overhead between stages. In the fused kernel, one thread advances a walk and constructs the required encodings while retaining private state across steps. The final outputs still satisfy the model's interface.

The preliminary result is approximately a twofold speedup in walk computation. It is not a twofold claim about complete model training.

The mechanism is familiar from the earlier graph work: move less intermediate data by reorganizing when computation happens. The new research question is how far fusion should go. Longer fused work can increase register pressure or reduce available parallelism. That makes fusion depth and batch size architectural choices worth studying, rather than assuming one large kernel will always be best.

## 24. Reconstruction and trajectory execution — 1:45

Battus begins with a different task: reconstructing diffusion states between sparse observations of a graph. Both observed endpoints constrain the missing steps.

The method replaces a more elaborate proposal-and-sampling pipeline with approximate per-vertex probabilities, forward–backward inference, and decoding. That gives us repeated graph passes that can map to CUDA. My question is which inference stages and intermediate state are necessary for useful reconstruction. Removing a stage changes the approximation, so reconstruction quality and graph-causal feasibility must be evaluated alongside execution time.

The trajectory project explores another response to dependent memory access. Sometimes walks do not share enough addresses for locality grouping to help. Independent trajectory state still lets us interleave work: issue work for B while A is waiting, then return to A when its data is available. The small timeline shows that opportunity, not a measured stall-free schedule.

The current prototype uses deterministic path coordinates, CPU interleaving, and a GPU implementation. Its accuracy needs its own analysis; deterministic paths are not automatically interchangeable with independent Monte Carlo walks.

These projects expand the program in two directions: simplifying the information an algorithm carries, and finding useful execution overlap when locality is limited.

## 25. An adaptive-scheduling project — 1:55

The long-term question is whether we can make these architecture-aware choices systematically, instead of rediscovering them manually for every irregular algorithm.

My first project would be an adaptive scheduler for trajectories. The hypothesis is that short observations of a batch can predict whether regrouping for locality or interleaving for latency overlap will repay its cost.

The table gives the proposed decisions. If many walks revisit the same vertices, grouping may improve adjacency reuse. If shared addresses are rare but independent trajectories spend time waiting, interleaving may help. If neither offers enough benefit, the policy should preserve the current order.

The result I would aim to demonstrate is lower total execution time than tuned static schedules on unseen graphs and sampling workloads. That includes observation, sorting, and coordination overhead. A faster kernel that needs expensive preparation would not by itself establish success.

The application must supply the legal freedom: dependencies between tasks, ownership of random state, and the required accuracy. The runtime cannot silently exchange an independent sampler for a deterministic approximation.

Graph walks and NeuralBloom provide concrete initial workloads. The next steps are representation selection and CPU–GPU placement, followed by sparse and dynamic graph problems, sampling-heavy ML, and scientific Monte Carlo. Each extension tests whether the observations and cost model transfer beyond the original graph kernels.

## 26. Closing — 0:30

Finding regularity meant sharing topology across samples, choosing compact representations with uniform updates, and arranging trajectories to improve reuse.

My research connects algorithm design to memory behavior and parallel execution. The next step is to predict which of these transformations will help and choose them automatically, while preserving the application's result and accuracy requirements.

Thank you. I look forward to your questions.
