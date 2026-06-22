# Quantum Reservoir Computing for DDoS Anomaly Detection

A hands-on, fully runnable study of **Quantum Reservoir Computing (QRC)** applied to **Distributed
Denial-of-Service (DDoS)** detection in network traffic. The project encodes a traffic window into a
fixed (untrained) quantum circuit, reads out its state as features, and feeds them to an unsupervised
anomaly detector — comparing this quantum reservoir, head-to-head and at equal size, against a
classical Echo State Network.


---

## Why reservoirs?

Training a variational circuit runs into the **barren plateau** problem:
gradients vanish exponentially as the circuit grows, and optimisation stalls. **Reservoir computing**
sidesteps this entirely: the circuit is **fixed and random**, never trained; only a light read-out is
used downstream. It needs no quantum optimisation, and its dynamics naturally carry **memory** of
recent windows — a good fit for traffic that evolves over time.

We compare two reservoirs of the **same size**, differing only in the underlying machine:

| | Classical reservoir | Quantum reservoir |
|---|---|---|
| Type | Echo State Network | Fixed random circuit |
| Update | `h(t) = tanh(W_in u(t) + W h(t−1))` | `\|ψ⟩ = U(features)\|0⟩`, read `⟨Z_i⟩` |
| Richness from | nonlinearity + recurrence | superposition + **entanglement** |
| Read-out | node activations `c_node_*` | qubit expectations `z_qubit_*` |

---


## Getting started

```bash
git clone https://github.com/niccolo-sfregola/quantum-anomalydetection-ddos.git
cd quantum-anomalydetection-ddos

# Python 3.10+ recommended
pip install numpy pandas scikit-learn qiskit matplotlib jupyter

jupyter lab QRC_tutorial_aqc.ipynb   # or: jupyter notebook
```

Run the notebook top to bottom. It reads the local `cleaned_dataset/option_2/` folder directly
(set `DATA_DIR` in the loading cell if your data lives elsewhere). 

---

## What the tutorial covers

1. Why **reservoir computing** sidesteps the training pains of recurrent networks.
2. How a **fixed quantum circuit** turns into a feature extractor with no trainable quantum parameters, and why that is attractive for **NISQ** devices.
3. How a window's source-IP distribution can be viewed as a **density matrix**
4. How to plug those features into a classical **unsupervised** anomaly detector trained *only on benign traffic*.

**Roadmap**

1. Load the cleaned data and understand the features.
2. Aggregate flows into **time windows** and visualise the attack windows.
3. Introduce the **IP-address density matrix** and see why its scalar summaries are classical.
4. Run the **classical reservoir** and the **quantum reservoir**, side by side, and identify exactly which features are *quantum*.



---


## Acknowledgements

Built for the **ETH Quantum Hackathon 2026** (1st place, team *Qool Quids*) and adapted as teaching
material for the LMU course *Applications of Quantum Computing*. Base traffic data: NF-UNSW-NB15.

