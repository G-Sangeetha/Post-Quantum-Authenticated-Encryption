# Post-Quantum Authenticated Encryption on Networked Raspberry Pi Hardware — Reproduction Materials

## Contents

| File | Description |
|---|---|
| `PQ-Authenticated-Encryption.ipynb` | Desktop prototype: construction, correctness/security test suite, hash-to-curve validation, timing side-channel analysis, parameter/signature sweeps, head-to-head and component-breakdown benchmarks |
| `pqae_updated.spthy` | Tamarin model: shared scaffold (B1/PQAE-SHA/PQAE) and the weaker standard-composition strawman (B0), 5 lemmas |
| `tamarin_output.txt` | Full Tamarin prover output (`--prove` proof script) for all 5 lemmas |
| `standard_fail_unforgeability.png` | Attack-trace graph for `standard_fails_unforgeability`, exported from Tamarin's interactive mode |
| `Standard_fails_replay.png` | Attack-trace graph for `standard_fails_replay`, exported from Tamarin's interactive mode |
| `scaffold_executable.png` | Witness trace for `scaffold_executable`, confirming an honest run reaches acceptance |
| `results_main_sender.json`, `results_main_receiver.json`, `results_baseline_sender.json`, `results_baseline_receiver.json` | IoT networked timing benchmark, $N=100$ per size, 16–1024 B, main construction and standard baseline |
| `ablation_results_desktop.json`, `ablation_results_pi.json` | Ablation study (B1 / PQAE-SHA / PQAE) timing and peak-memory data, both platforms |
| `energy_runsheet.csv` | Raw energy measurement log: per-repetition start/end power-meter readings (mWh) and timestamps for both boards, across all constructions, sizes, and the idle/meter-swap calibration runs |
| `energy_results_final.csv` | IoT energy measurement, 3 repetitions per construction per size, raw and net figures with dual-meter cross-check — derived directly from `energy_runsheet.csv` |

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

This text output, reproduced in full in `tamarin_output.txt`, is sufficient evidence for the two all-traces lemmas (`sender_unforgeability`, `replay_resistance`), whose proofs are branching case-split trees rather than a single trace. The three exists-trace lemmas additionally have a single witness graph each; these are not produced by `--prove` and must be exported from interactive mode:

```bash
tamarin-prover interactive pqae_updated.spthy
```

Open `http://localhost:3001`, select the `pqae_updated.spthy` theory, open each exists-trace lemma, run autoprove from the Actions menu if it has not already resolved, and export the rendered constraint graph. The three graphs included here (`tamarin_standard_forgery.png`, `tamarin_standard_replay.png`, `tamarin_scaffold_executable.png`) were produced this way and also appear in Appendix A of the manuscript.

## IoT deployment data

JSON files report per-size summary statistics (mean, standard deviation, median, min/max, 95% CI) over repeated trials from two networked Raspberry Pi 4 boards communicating over TCP.

Energy is recorded at two levels. `energy_runsheet.csv` is the raw log: each row is one timed repetition, giving the start and end accumulated-energy reading (mWh) and timestamp for both the sender and receiver boards' inline power meters, for every (construction, message size) pair, plus the idle-draw runs and the meter-swap calibration run used for the cross-check reported in the manuscript. `energy_results_final.csv` is derived from it directly: per-repetition energy is the end-minus-start mWh delta converted to mJ, divided by the trial count (`n_trials`) for per-operation energy, and divided by elapsed time for average power; idle draw (from the `idle` rows) is subtracted to give the net, workload-only figure alongside the raw total; mean, standard deviation, and 95% confidence intervals are then computed across the three repetitions per (construction, board, size) cell. Recomputing any cell of `energy_results_final.csv` from the corresponding rows of `energy_runsheet.csv` reproduces the reported value exactly.
