---
title: "SLURM: Half load on hyperthreads"
description: "I misconfigured SLURM, read the manual, and configured it properly instead."
date: 2025-09-29
tags: ["slurm", "rtfm"]
categories: ["psa"]
series: ["PSAs"]
---

I spent more time than I care to admit this afternoon trying to understand why SLURM was underutilising CPUs on my compute node.
Worse still, I had encountered and resolved the same problem several months ago, so I am putting it here in case it is useful to someone else (*i.e.* me, in another six months[^sam]).


### Behaviour

~50% system load, despite full CPU allocation according to `scontrol`[^scontrol]:

```
$ scontrol show node mynode
NodeName=mynode
   Sockets=2
   CoresPerSocket=40
   ThreadsPerCore=2
   CPUAlloc=160
   CPUEfctv=160
   CPUTot=160
   State=ALLOCATED
   CfgTRES=cpu=160
   AllocTRES=cpu=160
```

Half load with full allocation smells of a misunderstanding between physical and logical cores.
Sure enough, 160 CPUs are allocated but only 80 jobs are in the running state:

```
$ squeue -t R -h | wc -l
80
```


### Solution

The answer was right under my nose [in the manual that I had read without reading](https://slurm.schedmd.com/slurm.conf.html#OPT_SelectTypeParameters):

> The node's `Boards`, `Sockets`, `CoresPerSocket` and `ThreadsPerCore` may optionally be configured and result in job allocations which have improved locality; however doing so will prevent more than one job from being allocated on each core.

When configuring my node in `slurm.conf`, I had dutifully filled out these optional CPU topology parameters to provide as much information as possible to SLURM about the CPUs in my node; inadvertently triggering some allocation strategy that optimises for placement to the hyperthreaded-capable physical cores, rather than the logical cores.

It seems counter-intuitive that it is possible to set `SelectTypeParameters=CR_CPU` (logical cores as consumable) and `ThreadsPerCore=2` if SLURM just behaves as if you'd set `SelectTypeParameters=CR_Core` (which sets physical cores as consumable) instead; but I suppose `SelectTypeParameters` determines the accounting units used to schedule jobs across all nodes in the cluster, whereas `ThreadsPerCore` is used to independently constrain how jobs may be allocated to the compute resource on individual nodes.

In short, if you have a CPU capable of hyperthreading you *probably*[^probably] do not want to be setting any of the CPU topology parameters and should set `CPUs` directly instead:

That is:
```
NodeName=mynode State=IDLE ... CPUs=160
```

and not:
```
NodeName=mynode State=IDLE ... Sockets=2 CoresPerSocket=40 ThreadsPerCore=2
```

Now we're cooking:
```
$ squeue -t R -h | wc -l
160
```

Happy scheduling!

[^scontrol]: I've removed irrelevant scontrol output keys.
[^probably]: You wouldn't be reading this blog if you really did mean to schedule hyperthread-aware jobs.
[^sam]: Sam, if you're reading this again, you owe me a beer.
