# Post-Quantum Authenticated Encryption on Networked Raspberry Pi Hardware — Reproduction Materials

## Contents

| File | Description |
|---|---|
| `PQ-Authenticated-Encryption.ipynb` | Desktop prototype: construction, correctness/security test suite, hash-to-curve validation, timing side-channel analysis, parameter/signature sweeps, head-to-head and component-breakdown benchmarks |
| `pqae_updated.spthy` | Tamarin model: shared scaffold (B1/PQAE-SHA/PQAE) and the weaker standard-composition strawman (B0), 5 lemmas |
| `tamarin_output.html` | Full Tamarin prover output, including attack-trace graphs for the two exists-trace lemmas |
| `results_main_sender.json`, `results_main_receiver.json`, `results_baseline_sender.json`, `results_baseline_receiver.json` | IoT networked timing benchmark, $N=100$ per size, 16–1024 B, main construction and standard baseline |
| `ablation_results_desktop.json`, `ablation_results_pi.json` | Ablation study (B1 / PQAE-SHA / PQAE) timing and peak-memory data, both platforms |
| `energy_runsheet.xlsx` | IoT energy measurement, 3 repetitions per construction per size, raw and net figures with dual-meter cross-check |

## Reproducing the desktop results

Environment: Python 3.11, `liboqs-python`, `gmpy2`. Open `PQ-Authenticated-Encryption.ipynb` and run top to bottom.

## Reproducing the Tamarin verification

Environment: Tamarin Prover 1.8.0, Maude 3.1 (git revision `f172d7f00b1485446a1e7a42dc14623c2189cc42`, compiled 2023-09-01), tested on Ubuntu in a VirtualBox VM.

```bash
tamarin-prover --prove pqae_updated.spthy
```

Expected output:
```bash
summary of summaries:

analyzed: pqae_updated.spthy

processing time: 3.85s

sender_unforgeability (all-traces): verified (11 steps)
replay_resistance (all-traces): verified (12 steps)
standard_fails_unforgeability (exists-trace): verified (7 steps)
standard_fails_replay (exists-trace): verified (12 steps)
scaffold_executable (exists-trace): verified (13 steps)
```


To regenerate the full HTML output with attack-trace graphs:
```bash
tamarin-prover pqae_updated.spthy --prove --output=tamarin_output.html
```


## IoT deployment data

JSON files report per-size summary statistics (mean, standard deviation, median, min/max, 95% CI) over repeated trials from two networked Raspberry Pi 4 boards communicating over TCP. `energy_runsheet.xlsx` records three independent repetitions per (construction, size) pair, with dual inline power meter readings (raw and idle-subtracted net energy) and the meter cross-check run.

