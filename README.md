# Contrastive Learning for Rare Genetic Disease Classification

This repository contains the implementation, dataset preparation workflow, and experimental materials for the thesis project **“Contrastive Learning for Rare Genetic Disease Classification.”**

The project explores whether **self-supervised contrastive learning** can discover meaningful disease-related gene groups when labeled rare disease data are limited, fragmented, and biologically heterogeneous.

---

## Overview

Rare genetic disease classification is difficult because most rare diseases have very few labeled examples, phenotype descriptions are often incomplete, and relevant information is distributed across multiple biomedical sources. Traditional supervised machine learning models usually require large labeled datasets, which are not easily available in rare disease research.

To address this challenge, this project proposes a **SimCLR-style self-supervised contrastive learning framework** that learns gene embeddings without using disease labels during training. The learned embeddings are then clustered to identify biologically meaningful groups of rare disease-related genes.

---

## Key Idea

Instead of training the model with predefined disease classes, the system creates two augmented views of the same gene and teaches the model that both views represent the same biological entity.

```text
Same gene, two augmented views  →  pull closer
Different genes                 →  push apart
```

After training, each gene is represented as a compact embedding vector. Genes with similar phenotype patterns should appear closer together in the learned embedding space.

---

## Research Objectives

The main objectives of this thesis are:

* To integrate gene metadata, disease associations, and HPO phenotype terms from reliable biomedical sources.
* To convert heterogeneous biological data into fixed-size numerical feature vectors.
* To train a self-supervised contrastive learning model without relying on disease labels.
* To extract gene embeddings and cluster rare disease-related genes.
* To evaluate whether the discovered clusters reflect meaningful biological disease structure.

---

## Data Sources

The dataset was constructed from three public biomedical resources:

| Source                         | Purpose                                                                                        |
| ------------------------------ | ---------------------------------------------------------------------------------------------- |
| NCBI                           | Gene metadata such as official gene symbols, gene names, chromosome, gene type, and exon count |
| DisGeNET                       | Gene-disease association information                                                           |
| Human Phenotype Ontology (HPO) | Standardized clinical phenotype terms and identifiers                                          |

The final dataset contains:

| Attribute                          |  Value |
| ---------------------------------- | -----: |
| Rare disease-related genes         |    227 |
| Gene-disease association records   |    621 |
| Gene-phenotype association records | 13,835 |
| Unique disease names               |    548 |
| HPO TF-IDF features                |    800 |
| Organ-system score features        |     11 |
| Genomic structural features        |      3 |
| Total engineered features          |    814 |
| PCA-reduced input dimension        |    128 |
| Final embedding dimension          |     64 |
| Final number of clusters           |     18 |

---

## Methodology

The complete pipeline follows five main stages:

```text
Raw Biomedical Data
        ↓
Feature Engineering
        ↓
Self-Supervised Contrastive Training
        ↓
Embedding Extraction and Clustering
        ↓
Evaluation and Biological Interpretation
```

### 1. Dataset Construction

Gene metadata, disease associations, and phenotype associations were collected and matched using official gene symbols. Only genes available in both genotype and phenotype datasets were retained.

### 2. Feature Engineering

Each gene was represented using three feature groups:

```text
800 HPO TF-IDF features
+ 11 organ-system percentage score features
+ 3 genomic structural features
= 814-dimensional feature vector
```

The feature engineering process included:

* cleaning and normalizing dataset columns;
* matching genotype and phenotype records;
* normalizing disease names;
* converting HPO terms into a binary gene-phenotype matrix;
* filtering overly common and very rare phenotype terms;
* applying TF-IDF weighting;
* generating organ-system scores;
* encoding chromosome, gene type, and exon count;
* applying StandardScaler;
* applying organ-system feature boosting;
* reducing 814 dimensions to 128 dimensions using PCA.

### 3. Smart Gene Augmentation

To support contrastive learning, two augmented views of each gene were generated.

The final augmentation strategy used:

| Augmentation                      | Value |
| --------------------------------- | ----: |
| Feature dropout                   |   10% |
| Gaussian noise standard deviation |  0.08 |

This simulates incomplete phenotype reporting and natural noise in biomedical records.

### 4. Contrastive Learning Model

The model follows a SimCLR-style architecture:

```text
Input: 128-dimensional PCA feature vector
        ↓
Encoder Network
        ↓
h: 64-dimensional gene embedding
        ↓
Projection Head
        ↓
z: 32-dimensional projection vector
```

Important distinction:

* `h` is used for clustering and downstream analysis.
* `z` is used only for contrastive loss during training and discarded afterward.

### 5. Clustering

After training, 64-dimensional embeddings were extracted for all 227 genes.

The clustering process included:

* L2 normalization of embeddings;
* PCA on embeddings;
* grid search over cluster count values;
* comparison of KMeans and Agglomerative Clustering;
* final selection using penalized silhouette score.

The best final configuration was:

```text
Clustering method: Agglomerative Clustering
Linkage: Ward
Selected k: 18
```

---

## Model Architecture

The final model uses a compact encoder suitable for the small dataset size.

```text
Input: 128 dimensions

Encoder:
Linear → BatchNorm → ReLU → Dropout
Linear → BatchNorm → ReLU → Dropout
Linear

Encoder output:
h = 64-dimensional embedding

Projection Head:
Linear → BatchNorm → ReLU
Linear

Projection output:
z = 32-dimensional vector
```

The model was trained using **NT-Xent contrastive loss** for 300 epochs.

---

## Final Results

The final model achieved the following performance using HPO organ-system labels as the primary evaluation ground truth:

| Metric                        |         Value | Interpretation                                              |
| ----------------------------- | ------------: | ----------------------------------------------------------- |
| Adjusted Rand Index           |         0.407 | Meaningful agreement with biological ground truth           |
| Normalized Mutual Information | 0.476 / 0.564 | Moderate shared information between clusters and labels     |
| Homogeneity                   | 0.601 / 0.651 | Clusters are relatively pure                                |
| Completeness                  | 0.394 / 0.498 | Some disease categories are split across clusters           |
| Silhouette Score              | 0.128 / 0.435 | Moderate cluster separation depending on evaluation setting |
| Keyword-based ARI             |         0.227 | Lower due to noisy keyword-based labels                     |

The HPO organ-system evaluation produced stronger results than keyword-based evaluation, showing that phenotype-based labels are more reliable than disease-name keyword matching.

---

## Important Findings

* The final pipeline discovered biologically meaningful gene clusters without using disease labels during training.
* Some clusters showed strong disease-specific purity, including hearing-related and cardiac-related groups.
* The keyword-based evaluation was noisier because many disease names could not be cleanly assigned to a category.
* HPO organ-system labels provided a more reliable biological ground truth.
* Multi-system diseases such as Usher syndrome created natural disagreement between hearing and ocular categories.
* The final model showed that meaningful biological structure can be learned from small, fragmented, and label-scarce rare disease data.

---

## Why 18 Clusters but 10 True Labels?

The model produced 18 clusters because clustering was performed in an unsupervised way based on learned gene embeddings. The 10 true labels are broad HPO organ-system categories used only for evaluation.

A single broad disease category may contain multiple biological subtypes. Therefore, it is reasonable for the model to divide 10 broad disease labels into 18 finer clusters.

This explains why homogeneity is higher than completeness: many clusters are fairly pure, but some true disease categories are distributed across multiple clusters.


---

## Requirements

Main libraries used in this project include:

```text
Python
PyTorch
scikit-learn
pandas
NumPy
matplotlib
seaborn
UMAP
t-SNE
```

---

## How to Run

### 1. Prepare the Dataset

Place the genotype and phenotype datasets inside the `data/` folder.

```text
data/final_genotype_dataset.csv
data/final_phenotype_dataset.csv
```

### 2. Run Feature Engineering

```bash
python src/feature_engineering.py
```

This generates the 814-dimensional feature representation and PCA-reduced 128-dimensional input features.

### 3. Train the Contrastive Model

```bash
python src/train.py
```

The model trains using augmented gene views and NT-Xent contrastive loss.

### 4. Extract Embeddings

```bash
python src/evaluation.py
```

This extracts 64-dimensional gene embeddings from the encoder.

### 5. Perform Clustering

```bash
python src/clustering.py
```

This applies clustering and saves the final cluster assignments.

---

## Evaluation Metrics

The following metrics were used to evaluate clustering quality:

| Metric                        | Purpose                                                                |
| ----------------------------- | ---------------------------------------------------------------------- |
| Adjusted Rand Index           | Measures agreement between predicted clusters and biological labels    |
| Normalized Mutual Information | Measures shared information between clusters and labels                |
| Homogeneity                   | Measures whether each cluster contains mostly one disease type         |
| Completeness                  | Measures whether genes from the same disease type are grouped together |
| V-measure                     | Harmonic balance between homogeneity and completeness                  |
| Silhouette Score              | Measures cluster separation and cohesion                               |
| Davies-Bouldin Index          | Measures cluster overlap; lower is better                              |

---

## Limitations

The main limitations of this project are:

* The dataset contains only 227 genes.
* HPO annotations are uneven across diseases.
* Some diseases are multi-system and do not fit into a single category.
* Evaluation labels are not perfect clinical ground truth.
* The model has not been clinically validated.
* Protein interaction networks and pathway-level data were not included.

---

## Future Work

Future improvements may include:

* expanding the dataset to 500–1000+ genes;
* adding OMIM, ClinVar, and pathway databases;
* integrating protein-protein interaction networks;
* using graph neural networks;
* incorporating biological language models or protein sequence embeddings;
* validating clusters with clinical genetics experts;
* developing a phenotype-to-cluster query interface.

---

## References

1. Jaiswal, A. et al. A Survey on Contrastive Self-Supervised Learning. Technologies, 2021.
2. Chen, T. et al. A Simple Framework for Contrastive Learning of Visual Representations. ICML, 2020.
3. Kingma, D. P. and Ba, J. Adam: A Method for Stochastic Optimization. ICLR, 2014.
4. Taleb, A. et al. ContIG: Self-Supervised Multimodal Contrastive Learning for Medical Imaging with Genetics. CVPR, 2021.
5. Chen, Y. et al. CoGO: Contrastive Learning for Predicting Disease Similarity. Bioinformatics, 2022.
6. Roman-Naranjo, D. et al. Systematic Review of Machine Learning Approaches in Rare Disease Classification. Journal of Medical Internet Research, 2023.
7. National Center for Biotechnology Information. NCBI, 2024.
8. Human Phenotype Ontology Consortium. Human Phenotype Ontology, 2024.
9. DisGeNET. A Discovery Platform for Human Disease Genes and Variants, 2024.

---

## Disclaimer

This repository is intended for academic and research purposes only. The proposed model is not a medical diagnostic system and should not be used for clinical decision-making without expert validation and regulatory review.
