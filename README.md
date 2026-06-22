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

## Repository structure

```
.
├── QRC_tutorial_aqc.ipynb          # the guided tutorial (start here)
├── src/
│   ├── quantum_reservoir.py        # QuantumReservoir: encoding, entangling layers, read-out
│   └── ip_quantum_enrichment.py    # QuantumReservoirFeatureMap (IP-distribution enrichment)
├── cleaned_dataset/
│   └── option_2/
│       ├── normal/{train,validation,test}/    # *_clean.csv  +  *_audit.csv
│       └── attack/{train,validation,test}/    # *_clean.csv  +  *_audit.csv
└── README.md
```

*(Adjust the paths above to match your actual layout.)*

---

## The dataset

The data is based on the public **NF-UNSW-NB15** NetFlow dataset, with DDoS bursts seeded in, and is
shipped **already cleaned** under `cleaned_dataset/option_2/`. Each split comes as a pair of files:

- `*_clean.csv` — the preprocessed **per-flow features** (one row per network flow);
- `*_audit.csv` — the ground-truth labels kept *out* of the model (`Label`, `Attack`,
  `is_seeded_ddos`, `row_in_window`).

Cleaning applied upstream: heavy-tailed quantities (`duration`, `total_bytes`, `packet_size_avg`) are
log-compressed, the protocol is one-hot encoded, and flows are grouped into `window_id` **time
windows** (detection happens per window, not per flow).

Per-flow columns used downstream: `src_ip`, `dst_ip`, `duration`, `total_bytes`,
`packets_per_second`, `packet_size_avg`, `outbound_byte_ratio`, `window_id`.

> **Data provenance.** The base traffic comes from NF-UNSW-NB15; please respect its original terms and
> attribution when redistributing. The seeded DDoS labels and the cleaning pipeline are part of this
> project.

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
(set `DATA_DIR` in the loading cell if your data lives elsewhere). To keep the walkthrough light,
`MAX_FILES_PER_SPLIT` limits how many files are loaded — raise it to use more data.

---

## What the tutorial covers

A didactic, runnable walkthrough of the whole pipeline:

1. **Load** the cleaned traffic data and understand the per-flow features.
2. **Aggregate** flows into time windows and visualise the attack windows.
3. **Density matrix** of the source-IP distribution — and why its scalar summaries (entropy, distance)
   are, on a diagonal state, purely *classical*.
4. **Two reservoirs** implemented side by side — a classical Echo State Network and a quantum circuit.
5. **What the circuit really does** — a worked `ρ → UρU†` example showing that the global entropy and
   trace distance are *unitarily invariant* (unchanged), while genuine **entanglement** appears.
6. **Comparison** with confusion matrices, ROC / precision-recall curves, score distributions, a
   PCA + linear-probe view of the feature spaces, and a feature-group ablation.

It includes **two short fill-in tasks** (the quantum entangling layer, and the classical ESN update),
each with a collapsible **Hint** and **Solution**, so the notebook can double as an exercise sheet.

### Features produced

Both reservoirs share the same window aggregates and a classical distribution distance; only the
reservoir read-out differs.

- **Classical:** `c_node_0…9` (node activations) + `c_state_entropy` (spread of the activations) + `l1_distance`
- **Quantum:** `z_qubit_0…9` (expectations after entangling) + `s_entanglement` (entanglement entropy) + `l1_distance`

Here `l1_distance` is the total-variation distance of the window's source-IP distribution to a benign
baseline. The **only** feature with no classical counterpart is `s_entanglement`.

---

## Results (illustrative)

On this clean benchmark the task is easy: the distribution distance `l1_distance` alone already reaches
AUC ≈ 1.0, so the full-feature detector saturates for both methods. The interesting comparison is
therefore on the **reservoir features only**. Indicative single-run AUCs (small test split):

| feature group (alone) | detection AUC |
|---|---|
| classical `c_node_*` (10) | 0.834 |
| quantum `z_qubit_*` (10) | 0.978 |
| quantum `s_entanglement` (1) | 0.879 |
| classical `c_state_entropy` (1) | 0.519 |
| shared `l1_distance` (1) | 1.000 |

Two observations stand out: at equal size the quantum read-out is more discriminative
(`z_qubit_*` 0.978 vs `c_node_*` 0.834), and the **state-entropy scalar tells opposite stories** — the
quantum `s_entanglement` carries real signal (0.879) while its classical analogue `c_state_entropy` is
essentially chance (0.519). Same idea ("how complex is the reservoir's state"), different content.

---

## Honest framing & limitations

This is **not** a demonstration of quantum advantage:

- the dataset is **clean and easily separable** — simple classical statistics already flag every attack;
- the circuit is **classically simulated, noise-free** (no decoherence, the very effects that constrain
  real hardware);
- all figures are **single-run AUCs on a small test split** — treat them as indicative, not definitive.

What the experiments support is a **representational** claim: at equal size, the quantum reservoir
encodes a window more discriminatively than its classical counterpart, with a cleaner score margin.

## Next steps

The decisive test is the **data-scarcity regime** (detection AUC vs training-set size, averaged over
seeds), where the full project's experiments suggest the quantum methods hold up better with far less
data. Evaluation under realistic noise/decoherence models and on harder, traffic-masked attacks are the
other natural directions.

---

## Acknowledgements

Built for the **ETH Quantum Hackathon 2026** (1st place, team *Qool Quids*) and adapted as teaching
material for the LMU course *Applications of Quantum Computing*. Base traffic data: NF-UNSW-NB15.

## License

Add a license of your choice for the code (e.g. MIT). Note that the bundled data derives from
NF-UNSW-NB15 and is subject to that dataset's original terms — keep its attribution and licensing in
mind before redistributing.