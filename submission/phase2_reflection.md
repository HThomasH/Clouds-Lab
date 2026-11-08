# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->417.198<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.16<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->458341.2<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->13858.4<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4576<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2817.354 ms; 2 threads: 2061.491 ms; 4 threads: 2196.219 ms; 8 threads: 2288.857 ms. Two threads were fastest.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive version runs many loops in Python. Each iteration adds interpreter overhead and performs small operations. The vectorized version sends matrix operations to NumPy. NumPy uses OpenBLAS, which runs optimized compiled code and can use SIMD instructions. It processes data more efficiently and reduces Python overhead. Both versions compute the same attention. In my run, the time fell from 417.198 ms to 1.16 ms.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
Two threads were fastest at 2061.491 ms per step. Four threads took 2196.219 ms, and eight took 2288.857 ms. My Colab runtime reported two logical CPUs. More threads than available CPUs must share processor time. They also need to coordinate and share memory. These factors can explain the slowdown.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training uses large matrix operations in the forward and backward passes. A GPU can run many arithmetic operations in parallel, so it suits this workload. In my run, 100 steps took 458341.2 ms on the CPU and 13858.4 ms on the GPU. A tiny operation may not benefit: launching GPU work and moving data can cost more than the computation.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism stops helping when its overhead or a hardware limit outweighs its benefit. Vectorized attention took 1.16 ms instead of 417.198 ms. This shows the cost of Python loops in the naive version, but does not reveal a precise hardware limit. For threads, two were fastest at 2061.491 ms; four took 2196.219 ms. The runtime had two logical CPUs, so extra threads had to share CPU time and coordinate. These are likely reasons for the slowdown, although the sweep does not isolate them. For training, the GPU took 13858.4 ms for 100 steps, compared with 458341.2 ms on the CPU. The GPU accelerated this workload, but we did not vary its size or measure transfers separately. We therefore did not observe where GPU parallelism stopped helping. For small workloads, launch and data transfer overhead could become the limiting factors. Sequential work can also limit the overall speedup.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would repeat the CPU/GPU comparison with smaller batch sizes and separate data transfer time from computation. If the GPU remained much faster even for tiny batches, this would challenge my expectation that overhead reduces its advantage for small workloads.
<!--/answer-->
