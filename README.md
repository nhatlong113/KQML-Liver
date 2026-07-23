# KQML-Liver

## **Interpretable Hybrid Quantum Machine Learning with Kolmogorov–Arnold Networks for Liver Disease Classification**

> This paper is currently under review at **ACIVS 2026**, an **ICORE B-ranked conference**.

## **Introduction**

Liver disease remains a major global health concern, causing more than **two million deaths annually**. Early diagnosis is therefore essential, yet many existing **Machine Learning (ML)**, **Deep Learning (DL)**, and **Quantum Machine Learning (QML)** approaches still face several limitations:

- Limited interpretability in high-stakes clinical settings
- Reduced reliability on small and imbalanced medical datasets
- Inflated performance caused by data leakage or inconsistent evaluation protocols
- High computational requirements for purely quantum or deep architectures

To address these challenges, this paper proposes **KQML-Liver**, an interpretable hybrid framework for liver disease classification.

## **Proposed Method**

**KQML-Liver** integrates:

- **Kolmogorov–Arnold Networks (KANs)** for nonlinear feature learning and intrinsic interpretability
- A lightweight **Parameterized Quantum Circuit (PQC)** for modeling complementary higher-order feature interactions
- **Feature augmentation** using PCA, Factor Analysis, LDA, and a nonlinear trigonometric transformation
- **Effective Number of Samples (ENS)** weighting with power scaling and smoothing for class-imbalance handling
- **Focal Loss** to emphasize difficult training examples
- A strict **data-leakage-free evaluation protocol**

The complete framework first enriches the original clinical features, compresses them into a compact latent representation using KAN, and then processes the latent features using a shallow quantum circuit. The final prediction is generated through a sigmoid classifier.

<p align="center">
  <img src="assets/Fig1_Overall_Hybrid_KAN_Quantum_Architecture.png" width="850"/>
</p>

## **Dataset**

The model is evaluated on the **Indian Liver Patient Dataset (ILPD)** from the UCI Machine Learning Repository.

| Property | Value |
|---|---:|
| Number of records | 583 |
| Clinical input features | 10 |
| Positive liver-disease cases | 416 |
| Negative cases | 167 |
| Task | Binary classification |

The original class distribution is preserved, and no synthetic resampling is applied during evaluation.

## **Experimental Results**

Under a unified and leakage-free evaluation protocol, KQML-Liver achieves:

| Metric | Score |
|---|---:|
| **Accuracy** | **81.98%** |
| **Precision** | **80.20%** |
| **Recall** | **100.00%** |
| **F1-score** | **89.01%** |

The model successfully identifies all positive liver-disease cases in the test set, which is especially important in medical screening because false-negative predictions may delay diagnosis and treatment.

KQML-Liver also achieves competitive or superior performance compared with reproduced classical, hybrid, and quantum-learning baselines evaluated under the same protocol.

## **Main Contributions**

### **1. Feature Augmentation and Nonlinear Transformation**

A feature-enrichment strategy is introduced to overcome the limited representational capacity of the original ILPD feature space.

PCA, Factor Analysis, and LDA are applied in parallel to capture:

- Variance-based information
- Latent statistical structure
- Class-discriminative information

Their outputs are integrated with the original features and transformed using:

```math
x' = (x \cdot \cos(x)) \cdot \frac{\pi}{2}
```

This transformation enriches nonlinear structure and produces values suitable for quantum rotation-angle encoding.

<p align="center">
  <img src=""assets/Fig2_Feature_Augmentation_and_Nonlinear_Transformation_Pipeline.png" width="800"/>
</p>

### **2. Hybrid KAN–Quantum Architecture**

The proposed architecture combines the strengths of classical functional learning and quantum feature processing.

The complete end-to-end prediction pipeline is expressed as:

```math
\mathbf{z}=f_{\mathrm{KAN}}(\mathbf{x})
```

```math
\hat{y}=\mathrm{sigmoid}\left(g\left(U(\mathbf{z})\left|0\right\rangle^{\otimes m}\right)\right)
```

where the KAN transforms the enriched input vector into a compact latent representation, while the parameterized quantum circuit models complementary higher-order interactions.

#### **KAN-Based Feature Learning**

Unlike a conventional multilayer perceptron, which places fixed activation functions on nodes, KAN places learnable univariate functions on network edges.

The latent representation is produced by composing multiple KAN layers:

```math
\mathbf{z}
=
f_L \circ f_{L-1} \circ \cdots \circ f_1(\mathbf{x})
```

At layer l, the output of node j is computed by aggregating the transformed outputs from all nodes in the previous layer:

```math
h_j^{(l)}
=
\sum_{i=1}^{m_{l-1}}
\phi_{i,j}^{(l)}
\left(
h_i^{(l-1)}
\right)
```

where:

- `h_i^(l-1)` is the output of node i in the previous layer
- `phi_i,j^(l)` is a learnable spline-based function on the edge from node i to node j
- `m_(l-1)` is the number of nodes in the previous layer

This edge-level functional parameterization allows KAN to learn complex nonlinear relationships while preserving interpretability through the shapes of the learned spline functions.

<p align="center">
  <img src="Fig3_KAN_Module_Architecture.png" width="800"/>
</p>

#### **Quantum Feature Processing**

The compact KAN representation is encoded into an m-qubit quantum circuit.

First, all qubits are initialized in an equal superposition using Hadamard gates:

```math
\left|\psi_0\right\rangle
=
H^{\otimes m}\left|0\right\rangle^{\otimes m}
```

The QAOA-inspired quantum embedding alternates between a cost unitary and a mixer unitary:

```math
\left|\psi(\theta,\mathbf{z})\right\rangle
=
U_M(\beta_L)U_C(\gamma_L,\mathbf{z})
\cdots
U_M(\beta_1)U_C(\gamma_1,\mathbf{z})
\left|\psi_0\right\rangle
```

The cost Hamiltonian encodes both individual latent features and pairwise feature interactions:

```math
H_C
=
\sum_{i=1}^{m} z_i Z_i
+
\sum_{i=1}^{m-1}
\sum_{j=i+1}^{m}
J_{ij} Z_i Z_j
```

The mixer Hamiltonian promotes exploration of the quantum state space:

```math
H_M
=
\sum_i X_i
```

The corresponding parameterized unitaries are:

```math
U_C(\gamma_l)
=
e^{-i\gamma_l H_C},
\qquad
U_M(\beta_l)
=
e^{-i\beta_l H_M}
```

After the QAOA-inspired layers, CNOT gates introduce additional inter-qubit correlations. Each qubit is then measured in the Pauli-Z basis:

```math
\mathbf{z}_q
=
\left[
\langle Z_1\rangle,
\langle Z_2\rangle,
\ldots,
\langle Z_m\rangle
\right]
\in [-1,1]^m
```

The measured expectation vector is passed to the final sigmoid classifier:

```math
\hat{y}
=
\sigma\left(
\mathbf{w}^{T}\mathbf{z}_q+b
\right)
```

This design captures higher-order interactions while remaining compatible with the limited resources available in the **NISQ era**.

<p align="center">
  <img src="Fig4_Quantum_Module_Architecture.png" width="850"/>
</p>

### **3. Class-Imbalance Handling Without Synthetic Resampling**

KQML-Liver preserves the original ILPD data distribution and addresses imbalance using an enhanced **Effective Number of Samples (ENS)** strategy:

```math
w_c = \frac{1-\beta}{1-\beta^{n_c}}
```

The raw class weights are further refined using power scaling and smoothing:

```math
w_c \leftarrow w_c^\gamma
```

```math
w_c \leftarrow 1 + \alpha_{\mathrm{smooth}}(w_c - 1)
```

The resulting weights are normalized to unit mean, reducing excessive penalties while improving minority-class learning. Focal Loss is additionally used to focus training on difficult examples.

### **4. Intrinsic Model Interpretability**

The framework provides model transparency from three complementary perspectives:

- **Feature importance analysis** using perturbation, gradient sensitivity, and intrinsic KAN magnitude
- **Symbolic formula extraction** from learned KAN functions
- **Pairwise feature-interaction analysis**

The analysis identifies **Total Bilirubin (TB)** as the most influential feature, followed by the **Albumin/Globulin ratio**, **SGOT**, **Total Protein**, and **SGPT**.

The extracted symbolic representation can also support lightweight deployment because it can be evaluated without storing the full neural network or requiring a deep-learning framework.

<p align="center">
  <img src="Fig5_Feature_Interaction_Analysis.png" width="850"/>
</p>

## **Why KQML-Liver?**

KQML-Liver is designed to provide a practical balance between:

- Predictive performance
- Clinical sensitivity
- Model transparency
- Robustness to data leakage
- Small-data learning
- Quantum-resource efficiency

Rather than relying on a large quantum circuit, the KAN module first produces a compact and informative representation. The quantum component can therefore remain shallow and focus on modeling residual higher-order interactions.

## **Paper**

**Title:** KQML-Liver: Interpretable Hybrid Quantum Machine Learning with Kolmogorov–Arnold Networks for Liver Disease Classification

**Authors:**

- Nhat-Long Nguyen
- Viet-Loi Dinh
- Trong-Nghia Pham

**Affiliation:** Faculty of Information Technology, University of Science, Vietnam National University, Ho Chi Minh City, Vietnam

## **Citation**

The citation information will be updated after the paper is officially published.

```bibtex
@inproceedings{nguyen2026kqmlliver,
  title     = {KQML-Liver: Interpretable Hybrid Quantum Machine Learning with Kolmogorov--Arnold Networks for Liver Disease Classification},
  author    = {Nguyen, Nhat-Long and Dinh, Viet-Loi and Pham, Trong-Nghia},
  booktitle = {Proceedings of ACIVS 2026},
  year      = {2026},
  note      = {Under review}
}
```

## **Acknowledgments**

This research is supported by research funding from the **Faculty of Information Technology, University of Science, Vietnam National University – Ho Chi Minh City**.
