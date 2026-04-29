---
layout: post
title: When biology meets the cloud
subtitle: A look at inferring gene networks with parallel evolutionary algorithms
tags: [HPC, cloud computing, bioinformatics, algorithms]
author: Joana Ros Alonso
---

{: .box-note}
**Paper:** Lee, W.P., Hsiao, Y.T. and Hwang, W.C. (2014). Designing a parallel evolutionary algorithm for inferring gene networks on the cloud computing environment. *BMC Systems Biology*, 8:5.

Gene regulatory networks (GRNs) describe how genes interact with each other: which ones activate or suppress others, and how those relationships shape the behavior of living cells. Reconstructing these networks from experimental data is one of the core problems in systems biology, and it is also painfully slow to solve computationally. This paper takes a practical shot at that bottleneck, and the result is a good example of how cloud infrastructure can make certain complex scientific problems actually tractable.

## The problem with large gene networks

To model a gene network, the authors use the S-system model, a type of differential equation that captures how gene expression levels change over time. Inferring the network means finding all the parameters that best fit measured expression data, which quickly becomes a huge optimization problem as the number of genes grows.

Evolutionary algorithms are a natural fit for this kind of search: they explore the solution space in a somewhat random way and can avoid getting stuck in poor solutions. But they hit two walls when networks get large. First, the population tends to lose diversity too early and converges before finding a good answer. Second, computation time scales very badly.

## The hybrid GA-PSO approach

The authors propose a hybrid algorithm combining genetic algorithms (GA) and particle swarm optimization (PSO), called iGA-PSO. The idea is to get the best of both: PSO is good at local fine-tuning, GA at maintaining diversity through crossover and mutation. On top of that, they use a **parallel island model**: the population is split into smaller groups that evolve independently on separate compute nodes, with occasional migration of individuals between groups to share good solutions without losing diversity.

This island model is elegant not just algorithmically but also structurally. Each gene's subproblem maps naturally onto a separate island, which in turn maps naturally onto a separate compute node.

---

## Where cloud computing actually earns its place

This is the part worth paying attention to. The parallel island model is a solid technique, but running it on dedicated HPC clusters is expensive and out of reach for most research groups. The authors implement the whole thing on **Hadoop MapReduce**, which runs on regular hardware or cloud instances.

> Cloud computing is not just about raw speed here. It is about making certain classes of problems accessible to groups that cannot afford dedicated clusters.

MapReduce splits the work into map tasks and reduce tasks, with a master node coordinating everything and slave nodes doing the actual computation in parallel.

![General workflow of a MapReduce process](/images/paper_fig1.png){: .mx-auto.d-block :}

*Figure 1: General workflow of a MapReduce process, with mappers handling independent subtasks and reducers combining the results.*

The islands are arranged in a hypercube topology, so migration only happens between neighbouring nodes. Communication overhead stays low while good solutions still spread across the population over time.

![Parallel island model of the iGA-PSO approach](/images/paper_fig3.png){: .mx-auto.d-block :}

*Figure 3: Parallel island model of the iGA-PSO approach, with the hypercube communication topology on the right.*

This structure maps very cleanly onto MapReduce: map tasks run the local evolution per group, reduce tasks handle migration and combine results, and Hadoop takes care of data distribution and fault tolerance underneath. For a 125-gene network, sequential methods become impractical; distributing across cloud nodes makes it solvable in a reasonable time.

---

## Does it work?

The experiments test four datasets of yeast *S. cerevisiae* subnetworks (25, 50, 100, and 125 genes). The iGA-PSO consistently outperforms the plain GA-PSO, with better fitness values and lower variance, and the speedup from cloud distribution is real and substantial for larger networks.

{: .box-warning}
**That said:** the individual components (GA-PSO and island model) are well-established ideas. The actual contribution is the engineering integration, making these pieces work together on MapReduce. The paper reads more as a solid systems paper than an algorithmic breakthrough.

Still, the approach scales, it runs on standard infrastructure, and it actually recovers network behaviors that match biological data. For a field where wet lab experiments are slow and expensive, having a faster and cheaper computational inference pipeline, even an imperfect one, is genuinely valuable.
