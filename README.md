# Post-Quantum-Authenticated-Encryption
Implementation, verification, and benchmarks for the paper:

---

## Contents

| File | Description |
|---|---|
| `PQ-Authenticated-Encryption.ipynb` | All experiments, tests, benchmarks, and figures |
| `pqae.spthy` | Tamarin symbolic verification model |
| `results.json` | Pre-computed benchmark data for figure generation |

---

## Requirements

```bash
pip install liboqs-python scipy
```

[liboqs-python](https://openquantumsafe.org/) provides ML-KEM (FIPS 203),
ML-DSA (FIPS 204), and Falcon. If unavailable, every cell still runs
using insecure mock backends — mock results are for structural testing
only and are not reportable.

---

## Running the Notebook

```bash
jupyter notebook PQ-Authenticated-Encryption.ipynb
```

Run all cells top to bottom (**Kernel → Restart & Run All**).
The results-save cell writes `results.json`; the final part reads it
to generate all publication figures.

---

## Reproducing the Paper's Results

| Experiment | Trials | Expected output |
|---|---|---|
| Distinguisher | N = 10,000 | Naive ~75.2% accuracy, Fixed ~49.7% |
| Sweep A / B benchmarks | N = 100 per config | Section 4.4 tables |
| Head-to-head comparison | N = 1,000 | 3.9× to 97.0× speedup |
| Ratchet tests (T-R1–T-R5) | — | All PASS |
| Side-channel analysis | N = 10,000 per group | Cohen's d < 0.2 all 7 tests |

Results were obtained on Intel i5 11th Gen, 16 GB RAM, Windows 10,
Python 3.11.8. Timing will vary by hardware.

---

## Tamarin Verification

Install [Tamarin 1.8.0](https://github.com/tamarin-prover/tamarin-prover)
and Maude 3.1, then run:

```bash
tamarin-prover pqae.spthy --prove
```

Expected output:
```bash
sender_unforgeability (all-traces): verified (11 steps)
replay_resistance (all-traces): verified (12 steps)
standard_fails_unforgeability (exists-trace): verified (7 steps)
standard_fails_replay (exists-trace): verified (12 steps)
processing time: 3.26s
```

Lemmas 1–2 verify the proposed construction.
Lemmas 3–4 confirm attack traces against the standard
ML-KEM+HKDF+AEAD composition.

---

## Security Notice

`MockKEM` and `MockSig` are **insecure by design** — for structural
testing only. Never use mock backends outside the test suite.
