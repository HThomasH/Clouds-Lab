# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
50
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->252.0<!--/answer--> | <!--answer:vm_warm_median_ms-->212.5<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->2377.7<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->635.7<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->688.6<!--/answer--> | <!--answer:function_warm_median_ms-->416.7<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
Warm median latency, as printed by `measure.py`:

```text
VM           212.5 ms  #############
Cloud Run    635.7 ms  ########################################
Function     416.7 ms  ##########################
```
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM does not create new instances automatically, it uses one e2-micro machine. Its capacity is limited by that machine and its container configuration. Cloud Run can start new container instances when demand increases. I set a maximum of three instances, and it can scale down to zero when idle. The Cloud Function can also start new instances when demand increases. I set a maximum of three instances, it can also scale down to zero.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
For the VM, I created a VPC, a subnet, firewall rules and a virtual machine. I installed Docker, configured access to Artifact Registry, and started the container. I also had to manage the machine and its network access. For Cloud Run, I deployed the container image from Artifact Registry. I configured the region, port and instance limits. Google managed the servers and instances. For the Cloud Function, I deployed the source code and selected the Python runtime and entry point. Google built and deployed the function. I did not manage a VM.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
```text
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-221941546943.us-central1.run.app
LAST DEPLOYED BY: service-221941546943@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-11-03T21:43:23.038967Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-221941546943.us-central1.run.app
LAST DEPLOYED BY: thomas.hardy@eurecom.fr
LAST DEPLOYED AT: 2026-11-03T21:39:35.950456Z
```
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The list shows two Cloud Run services: `echo` and `echo-bot`. The `echo` service was created for my second-generation Cloud Function. The function therefore runs on Cloud Run infrastructure. The two services use the same underlying platform and can both scale down to zero.
The deployment paths are different. For `echo-bot`, I provided a container image. For the function, I provided source code and an entry point. Google built and deployed it.
My cold measurements were not close: Cloud Run took 2377.7 ms and the function took 688.6 ms. Each result is a single sample. These measurements are not enough to conclude that one platform is always faster.
<!--/answer-->
