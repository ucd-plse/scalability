# Understanding and Detecting Scalability Faults

This is the artifact of the submitted manuscript *Understanding and Detecting Scalability Faults in Large-Scale Distributed Systems*. This artifact contains two parts: (1) raw data description and (2) experiment reproduction.

Source code for ScaleLens can be found [here](https://github.com/ucd-plse/scalelens) and the Docker images for experiments can be found [here](https://hub.docker.com/r/ucdavisplse/scalelens/tags).

**Quick links:** [empirical data](#11-scalability-fault-anti-patterns-section-2) | [evaluation data](#12-evaluation-data-section-4) | [mini-scale experiments](#21-mini-end-to-end-experiment) | [full-scale experiments](#22-full-scale-experiments) | [ablation analysis](#23-run-ablation-analysis)

## 1. Raw Data Description  

This section provides scripts to generate all data presented in paper. Python package `pandas` is required ([install it](https://pandas.pydata.org/docs/getting_started/install.html)).

### 1.1 Scalability Fault Anti-Patterns (Section 2)

This section presents the raw data used for investigating the anti-patterns of scalability faults. In this section, 444 scalability faults were analyzed and categorized into 4 root-cause categories and 11 anti-patterns.

A CSV file that contains all 444 scalability faults, as well as tags indicating their anti-patterns, can be found [here](fault-study/scalability-bugs.csv).

The scripts below run from a clone of this repository:

``` sh
git clone https://github.com/ucd-plse/scalability
cd scalability
```

To generate table 1 in the paper, run the following command:

``` sh
REPO_DIR=$(git rev-parse --show-toplevel)
python $REPO_DIR/scripts/table-1-breakdown.py
```

To generate all data for the anti-patterns in empirical study, run the following command:

``` sh
REPO_DIR=$(git rev-parse --show-toplevel)
python $REPO_DIR/scripts/pattern-analysis.py
```

### 1.2 Evaluation Data (Section 4)

In this section, we describe the raw data from our evaluation. For RQ1, we provide raw outputs from ScaleLens and ScaleCheck. For RQ2, we provide our analysis details. 


#### 1.2.1 Research Question 1: Effectiveness of ScaleLens and Comparison with Baseline

Raw outputs from ScaleLens are available [here](evaluation-data/scalelens-outputs). There are one json file for each system, listing all DCFs, their dimensions with relationship, and the anti-patterns they are associated with. 

ScaleCheck is our baseline to compare with ScaleLens. Raw outputs from ScaleCheck are available [here](evaluation-data/scalecheck-outputs). For each system, there is one file containing the list of DCFs detected by ScaleCheck.

#### 1.2.3 Research Question 2: Result Analysis

We provide our ablation analysis of the results from ScaleLens on the latest versions of Cassandra, HDFS, and Ignite. The analysis details are available [here](evaluation-data/ablation-study).



## 2. Experiment Reproduction

This section provides instructions to reproduce the experiments conducted in the study. The ScaleLens pipeline ([source code](https://github.com/ucd-plse/scalelens)) is shipped as Docker images on Docker Hub ([link](https://hub.docker.com/r/ucdavisplse/scalelens/tags)); each image bundles the JAR, scripts, workloads, and the target system source so no further assembly is required.

### 2.1 Mini End-to-End Experiment

The mini experiments are intended as a quick way to exercise the full ScaleLens pipeline end-to-end on a single machine. We recommend running a mini experiment as a smoke test before launching the larger experiments in [§2.2](#22-full-scale-experiments), to confirm that ScaleLens behaves as expected in your environment.

We provide three self-contained Docker images:

- `ucdavisplse/scalelens:CA-3.11.0-mini` for CASSANDRA-3.11.0 (`add-node` workload)
- `ucdavisplse/scalelens:HD-3.1.0-mini` for HDFS-3.1.0 (`add-dn-block` workload)
- `ucdavisplse/scalelens:IG-2.8.0-mini` for IGNITE-2.8.0 (`add-node` workload)

The mini experiments have been tested on a 32 GB / 8-core Ubuntu 20.04 host, where each run completes in approximately 20 minutes ([video](https://github.com/ucd-plse/scalability/releases/tag/replication-package)). Other platforms might also work but have not been validated.

To run a mini experiment, start the container from one of the three images. For CASSANDRA-3.11.0:

``` sh
docker run -it --pull always ucdavisplse/scalelens:CA-3.11.0-mini
```

This drops you into a shell at `/home/scaleview/scaleview-core` inside the container. From inside the container, run:

``` sh
bash run.sh ./experiments/CA-3.11.0.yaml CA-3.11.0-workspace
```

After the experiment completes, the results are in `CA-3.11.0-workspace/` inside the container:

- `sdeps.json` — DCFs detected by ScaleView, with their correlated dimensions
- `sdeps-patterns.json` — anti-pattern labels assigned by ScalePick
- `statistics.txt` — summary counts

<details>
<summary>Run with <code>HD-3.1.0-mini</code> instead</summary>

``` sh
docker run -it --pull always ucdavisplse/scalelens:HD-3.1.0-mini
```

From inside the container, run:

``` sh
bash run.sh ./experiments/HD-3.1.0.yaml HD-3.1.0-workspace
```

Outputs land in `HD-3.1.0-workspace/`.

</details>

<details>
<summary>Run with <code>IG-2.8.0-mini</code> instead</summary>

``` sh
docker run -it --pull always ucdavisplse/scalelens:IG-2.8.0-mini
```

From inside the container, run:

``` sh
bash run.sh ./experiments/IG-2.8.0.yaml IG-2.8.0-workspace
```

Outputs land in `IG-2.8.0-workspace/`.

</details>

### 2.2 Full-Scale Experiments

Each `SYSTEM-VERSION` studied in the paper is also published as a full image. To run an experiment, for example with Cassandra 4.1.0, start the image:

``` sh
docker run -it --pull always ucdavisplse/scalelens:CA-4.1.0
```

You will be dropped into a shell at `/home/scaleview/scaleview-core`. From inside the container, run the experiment:

``` sh
bash run.sh experiments/CA-4.1.0.yaml workspace
```

To run other experiments, replace `CA-4.1.0` with any experiment ID (`SYSTEM-VERSION`) listed in the `experiments` directory [here](https://github.com/ucd-plse/scalelens/tree/master/experiments). Outputs are written to `workspace/` inside the container; full-scale runs require substantially more memory and time than the mini variants in [§2.1](#21-mini-end-to-end-experiment).

### 2.3 Run Ablation Analysis

The ablation analysis (Table III) is reproduced from the static evaluation data shipped in this repository, so the only setup is a clone:

``` sh
git clone https://github.com/ucd-plse/scalability
cd scalability
```

To list statistics of all fragments without applying ScaleView:

``` sh
REPO_DIR=$(git rev-parse --show-toplevel)
cd $REPO_DIR/evaluation-data/ablation-study/all-fragments && python run_stat.py
```

To see how ScaleLens narrows down to DCFs:

``` sh
REPO_DIR=$(git rev-parse --show-toplevel)
cd $REPO_DIR/evaluation-data/ablation-study/scalelens && python run_stat.py
```
