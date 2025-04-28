# PCAP Analysis

This project aims to automate the process of identifying the most suitable deployment model for a workload when migrating solutions from an existing data center to a hybrid cloud. We employ unsupervised learning techniques to analyze PCAP (Packet Capture) files, which contain network traffic data, to extract logical identifiers that help define the right deployment model.

## Workflow

1. **Data Collection** Capture relevant PCAP data from the data center network.
2. **Feature Extraction** Extract meaningful features from the network flows or packets. [More Details](https://github.com/torstenboettjer/pcap-analysis/blob/main/docs/physical_topology.md)
3. **Data Preprocessing** Clean and preprocess the extracted features (e.g., scaling, normalization) to ensure that algorithms work effectively.
4. **Model Selection and Training** Choose appropriate unsupervised learning algorithms based on your goals (clustering, dimensionality reduction, anomaly detection). Train the models on the preprocessed data.
5. **Result Interpretation** Analyze the output of the unsupervised learning algorithms (clusters, reduced dimensions, anomalies). This often involves visualizing the results and examining the characteristics of the identified patterns.
6. **Validation and Refinement** Evaluate the quality of the discovered patterns (e.g., using domain knowledge or by looking for consistency). Refine the feature extraction and model parameters as needed.

The analysis focuses on the IP protocol, a fundamental Layer 3 protocol, to understand network topology, communication patterns, and the identities of communicating devices. The project utilizes tools like Wireshark for packet analysis and Python for implementing unsupervised learning algorithms. The following techniques are employed to analyze the PCAP data:

### Clustering

As a first step, we group network flows or connections based on their similarities without knowing the application types beforehand. We extract various features from the PCAP data at the flow level (a flow is typically defined by a 5-tuple: source IP, destination IP, source port, destination port, protocol) and potentially packet-level statistics. Examples of features include:
* Number of packets in the flow
* Total bytes transferred in the flow
* Flow duration
* Packets per second
* Bytes per second
* Inter-arrival times of packets
* TCP flags (SYN, ACK, FIN, RST)  
* Distribution of packet sizes

#### Algorithms
* **K-Means** Groups flows into a predefined number of clusters based on minimizing the within-cluster variance, found by experimenting with the number of clusters.
* **Hierarchical Clustering** Creates a hierarchy of clusters, allowing us to explore patterns at different levels of granularity.
* **DBSCAN** Density-Based Spatial Clustering of Applications with Noise. It can discover clusters of arbitrary shapes and identify outliers (anomalous communication).

After clustering, we analyze the characteristics of each cluster. For example, one cluster might consist of flows with short durations and small packet sizes, potentially indicating control traffic. Another cluster might have long durations and large byte counts, suggesting data transfer. By examining the IP addresses and port numbers within each cluster, we start to infer which applications might be involved.

#### Dimensionality Reduction:

In a second step, we reduce the number of features extracted from the PCAP data while preserving the most important information. This helps to visualize complex communication patterns in a lower-dimensional space and make clustering more effective.

#### Algorithms
* **Principal Component Analysis (PCA)** Finds the principal components (directions of maximum variance) in the data.
* **t-distributed Stochastic Neighbor Embedding (t-SNE)** A non-linear dimensionality reduction technique particularly good at visualizing high-dimensional data in 2D or 3D, often revealing natural groupings.
* **UMAP (Uniform Manifold Approximation and Projection)** Another non-linear technique that often produces better global structure preservation than t-SNE and can be faster.
* **Interpretation** Visualizing the reduced data can reveal clusters or distinct groups of communication patterns that might not be obvious in the high-dimensional feature space. You can then investigate the original features of these groups to understand the underlying communication characteristics.  

### Anomaly Detection

As part of the analysis we identify unusual or unexpected communication patterns that deviate significantly from the norm. This can help detect security breaches, misconfigurations, or performance issues.

#### Algorithms
* **Isolation Forest** Identifies anomalies by isolating instances that are "easy" to separate from the rest of the data.
* **One-Class SVM** Learns a boundary around the "normal" data and identifies instances outside this boundary as anomalies.
* **Autoencoders (Neural Networks)** Train a neural network to reconstruct the input data. Anomalous data points will have a higher reconstruction error.

Detected anomalies highlight unusual communication flows that warrant further investigation. By examining the features of these anomalous flows (e.g., unusual ports, high traffic volume to a specific IP), we gain insights into potential issues or novel application interactions.  

## Workbench
Analyse PCAP files to identify deployment artifact types in a data center network. Main programming language is Python, virtual environments rely on [automatic shell activation](https://devenv.sh/automatic-shell-activation/) using devenv.sh and direnv.

### Initial Setup

It is recommended to prepare a python repository on github and clone the rope into a local folder. Within the local folder prepare the devevlopment environment

```sh
devenv init
```

Add the python environment parameter to `devenv.nix`

```nix
  # Python environment
  languages.python = {
    enable = true;
    # https://devenv.sh/languages/python/
    # packages = [ pkgs.python39Packages.pip ];
    venv.enable = true;
    venv.requirements = ''
      requests
      # torch
    '';
    # venv.packages = [ pkgs.python39Packages.pip ];
    # venv.packages = [ pkgs.python39Packages.pip ];
    uv.enable = true;
  };
```

After saving the devenv configuration python and uv should be available.

```sh
python --version && uv --version
```

We add the dotfiles for the virtual environment to `.gitignore`

```sh
echo ".envrc" >> .gitignore
```

### Start analyzing

Before loading data, we initilaise the virtual environment

```sh
uv init && uv venv
source .devenv/state/venv/bin/activate
```

## Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
