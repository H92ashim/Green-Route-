# Green-Route — Supplementary Code, Calibration Data, and Reproducibility Materials

**Supplementary materials for:** *Green-Route: A Calibrated Carbon- and Service-Aware Routing Framework*

This repository provides the source code, benchmark measurements, calibration artifacts, simulated experiment results, figures, and manuscript-ready tables supporting the Green-Route study. It is intended for **public scholarly sharing**, **transparent evaluation**, and **reproducibility**. Readers can inspect the supplied results and rebuild visualizations without repeating the original hardware benchmark or full simulation.

> **Scope:** This is a research artifact, not a production-ready routing service. The modelled service-adequacy probabilities, request-demand scenarios, and hosted-tier assumptions are simulation inputs; they must not be interpreted as independent field measurements or guarantees of real-world performance.

## Contents at a glance

| Component | What it contains |
| --- | --- |
| [`green_route_study.py`](green_route_study.py) | Benchmarking, calibration processing, Green-Route and baseline simulations, sensitivity analysis, and result generation. |
| [`verify_study.py`](verify_study.py) | Checks for benchmark structure, routing invariants, simulation consistency, stored figures/tables, and the companion LaTeX manuscript. |
| [`regenerate_figures.py`](regenerate_figures.py) | Rebuilds manuscript figures and tables **from saved outputs**, without rerunning simulations. |
| [`tourism_benchmark_prompts.json`](tourism_benchmark_prompts.json) | Controlled set of 30 tourism and heritage-assistance prompts: 10 simple, 10 moderate, and 10 complex. |
| [`calibration/`](calibration/) | Benchmark observations, fitted calibration, metadata, summary CSVs, idle-power measurements, and raw Apple `powermetrics` logs. |
| [`study_outputs/`](study_outputs/) | Stored simulation results, parameter draws, scenario and strategy summaries, sensitivity outputs, break-even analysis, figures, and LaTeX assets. |
| [`requirements.txt`](requirements.txt) | Pinned Python scientific-computing dependencies. |
| [`README_SUPPLEMENTARY.md`](README_SUPPLEMENTARY.md) | Original revision-specific notes distributed with the supplementary archive. |

<details>
<summary>Repository layout</summary>

```text
.
├── README.md
├── README_SUPPLEMENTARY.md
├── requirements.txt
├── green_route_study.py
├── regenerate_figures.py
├── verify_study.py
├── tourism_benchmark_prompts.json
├── calibration/
│   ├── benchmark_metadata.json
│   ├── benchmark_raw.csv
│   ├── benchmark_run.log
│   ├── calibration.json
│   ├── calibration_models.csv
│   ├── calibration_summary.csv
│   ├── idle_power_samples.csv
│   └── power_logs/                 # idle + model repetition .plist logs
└── study_outputs/
    ├── simulation_metadata.json
    ├── parameter_draws.csv
    ├── simulation_results.csv
    ├── simulation_summary.csv
    ├── break_even_analysis.csv
    ├── envelope_comparison.csv
    ├── operating_regimes.csv
    ├── *_values.tex
    ├── adequacy_forms/            # per-form summaries and rank stability
    ├── figures/                  # Fig1–Fig6, each in PNG/PDF/EPS
    └── tables/                   # CSV and manuscript-ready LaTeX tables
```

</details>

## Research workflow

The study combines a **three-tier local language-model benchmark** with a **calibrated, paired stochastic simulation** comparing Green-Route with alternative routing strategies.

- **Benchmark tiers:** compact (`tinyllama:latest`), mid (`llama3.2:3b`), and large (`llama3.1:8b`). Each tier was evaluated on 30 prompts over five repetitions, resulting in 450 benchmark observations.
- **Recorded hardware setup:** Apple M1 Pro (16 GB RAM), macOS, Ollama, and Apple `powermetrics`; the full environment and model identifiers are recorded in [`calibration/benchmark_metadata.json`](calibration/benchmark_metadata.json).
- **Simulation scenarios:** off-peak, peak season, festival surge, and safety disruption.
- **Compared policies:** large-only, mid-default, fixed-threshold, quality cascade, Green-Route, and a perfect-information oracle. The oracle is a reference comparator, not a deployable policy.
- **Stored simulation configuration:** 200 parameter draws × 10 replications × four scenarios × six policies (48,000 scenario–policy–draw–replication rows), with seed 42 and 15-minute intensity bins; see [`study_outputs/simulation_metadata.json`](study_outputs/simulation_metadata.json).

The experiments evaluate modelled service adequacy, latency/queue-related behavior, energy use, and operational/extended carbon accounting under the study's stated system boundaries. Strategies share paired simulation inputs for comparison.

## Getting started

**Requirements:** Python **3.11+** and a supported Python environment. The provided results, visualization regeneration, and sensitivity-summary analysis do **not** require Ollama or Apple hardware.

```bash
# Run in the repository root
python -m venv .venv
source .venv/bin/activate       # Windows PowerShell: .venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

The dependency versions are pinned for consistency. Run all commands below from the **repository root** so relative input paths resolve correctly.

### 1. Inspect existing results (no computation required)

Start with [`study_outputs/simulation_summary.csv`](study_outputs/simulation_summary.csv), [`study_outputs/figures/`](study_outputs/figures/), and [`calibration/calibration_summary.csv`](calibration/calibration_summary.csv). The accompanying metadata JSON files describe the recorded run configuration.

### 2. Rebuild the figures and manuscript tables (recommended)

```bash
python regenerate_figures.py
```

This command reads the saved calibration, simulation results, summary, and parameter draws. It regenerates `study_outputs/figures/Fig1`–`Fig6` in PNG, PDF, and EPS formats, together with derived tables and LaTeX macros. **It does not rerun the benchmark or stochastic simulation.** It writes into `study_outputs/`, so make a copy first if you wish to keep byte-for-byte original presentation artifacts.

### 3. Rebuild adequacy-form ranking summaries

```bash
python green_route_study.py form-stability
```

This recomputes rank-stability outputs from the archived alternative adequacy-form summaries, without rerunning each alternative-form simulation. Results are written in `study_outputs/adequacy_forms/` and associated LaTeX files.

### 4. Run the complete baseline simulation (optional; compute-intensive)

```bash
python green_route_study.py simulate \
  --draws 200 --replications 10 --seed 42 \
  --workers 4 --output-dir reproduced_outputs
```

This performs a new simulation using the supplied calibration. Output is written to `reproduced_outputs/` to avoid overwriting the archived study results. Worker count can be adjusted for your machine. Note that elapsed time, parallel execution, or software/hardware differences can affect practical reproducibility; compare the generated CSV results and metadata with the archived artifacts before drawing conclusions.

To rerun the full alternative-form sensitivity study, use:

```bash
python green_route_study.py adequacy-form-sweep \
  --draws 200 --replications 10 --seed 42 \
  --workers 4 --output-dir reproduced_sensitivity
```

**Archive coverage:** The package retains the alternative-form **summary** and **rank-stability** files, but not each alternative form's per-realization records. Those records require rerunning `adequacy-form-sweep`.

### 5. Verify the archive and accompanying manuscript (optional)

```bash
python verify_study.py --paper /path/to/manuscript.tex
```

The verifier checks input definitions, stored benchmark/calibration files, model and policy invariants, selected paired simulation recomputation, figure/table presence, and manuscript-specific LaTeX structure. **The manuscript source file is not included in this supplementary ZIP.** Verification therefore requires the matching manuscript source, with its referenced `study_outputs/` LaTeX macro files available alongside it. This is a revision-specific verification suite, not a universal check for arbitrary papers; running it without the matching manuscript is expected to fail.

## Repeating the hardware benchmark (optional; macOS-specific)

The original energy measurements use Apple `powermetrics` and an Ollama server. This step is **not needed** for reviewing or replotting the archived findings. For a new measurement run on compatible macOS hardware:

```bash
# Install/start Ollama separately and obtain the required local models
ollama pull tinyllama:latest
ollama pull llama3.2:3b
ollama pull llama3.1:8b

# Start the Ollama server if it is not already running
ollama serve

# In another terminal, authorize macOS power sampling
sudo -v
python green_route_study.py benchmark --output-dir reproduced_calibration
```

The benchmark's `powermetrics` sampler requires elevated privileges and macOS support for the requested CPU/GPU metrics. Actual model versions, thermal state, power settings, background workloads, and device architecture can change measured energy and latency. New measurements must be treated as a **new calibration**, not as identical to the supplied Apple M1 Pro observations. See `green_route_study.py benchmark --help` for command options.

## Reading and interpreting the data

| Files | Interpretation |
| --- | --- |
| `calibration/benchmark_raw.csv` | Per-prompt inference timing, tokens, power-derived energy, and generated responses. |
| `calibration/calibration.json` | Fitted tier/complexity calibration parameters used by the simulator. |
| `study_outputs/parameter_draws.csv` | Seeded uncertainty input draws. |
| `study_outputs/simulation_results.csv` | Per-scenario, per-policy, per-draw, per-replication results. |
| `study_outputs/simulation_summary.csv` | Aggregated policy and scenario summaries for comparison. |
| `study_outputs/break_even_analysis.csv` | Derived break-even analysis from the simulated results. |
| `study_outputs/adequacy_forms/` | Sensitivity of comparisons to alternative service-adequacy functional forms. |
| `study_outputs/tables/` and `study_outputs/figures/` | Ready-to-reference tabular and visual results for the manuscript. |

**Interpretation cautions:**

1. **Measured vs. modelled:** Benchmark energy/timing is local-device measurement; service adequacy is a synthetic probabilistic model rather than independently graded response quality. Simulated workloads, hosted-tier roles, and carbon-intensity assumptions are not direct field measurements.
2. **System boundaries:** The operational accounting includes measured dynamic inference energy, PUE, network energy, and allocated edge idle power. The extended boundary additionally includes amortized edge hardware emissions. Consult the manuscript before comparing results with differently defined carbon accounting.
3. **Transferability:** Results are conditional on the recorded model snapshots, hardware calibration, uncertainty assumptions, and workload definitions. They do not automatically transfer to production deployments, other hardware, or different application domains.
4. **Reproducibility levels:** Rebuilding figures from saved results, regenerating a paired simulation, repeating the entire stochastic experiment, and recapturing power traces are different tasks with different requirements.

## Citation

If these materials support your work, **cite the corresponding Green-Route manuscript** and identify this repository or supplementary archive as the source of the supporting code and data. The ZIP does not provide verified author, publication, DOI, or permanent repository identifiers; add the authoritative citation and archival link here when available.

Suggested data-availability wording for a manuscript or repository description:

> Supplementary source code, benchmark and power measurements, calibration files, simulated results, and figure-generation scripts for the Green-Route study are provided in this repository. The archive includes metadata describing the experimental configuration and the limitations of rerunning hardware-specific measurements.

## Licensing and responsible reuse

**No license file is supplied in the source archive.** Public visibility on GitHub does not by itself establish permission to reuse, modify, or redistribute code or data. The maintainers should add an explicit `LICENSE` (and, if appropriate, a separate data license) before inviting unrestricted reuse. Dependencies and third-party model weights retain their own applicable terms; model weights are **not** distributed in this archive.

For scientific use, retain provenance and distinguish measured quantities from modelled assumptions. Please do not represent the supplied simulated outputs as field observations or claim production safety assurances from them.

## Questions, issues, and contributions

Use the repository's **Issues** section to report reproducibility problems, request missing context, or suggest documentation improvements. When reporting an issue, include your Python version, operating system, command, relevant input/output file, and a minimal error trace. For substantial scientific changes, describe how the new configuration differs from the archived study.

---

*Archive provenance: the supplied supplementary package contains an earlier `README_SUPPLEMENTARY.md` noting a September 2026 revision. The original benchmark observations and saved simulation outcomes are treated as archived research records; this GitHub README does not modify them.*
