# A False Average

Code and data for *A False Average: Pooled CoT-Monitor Accuracy Conceals a Reasoning-Dependent
Fragility* (Shikhar Shiromani and Leo Richter; NeurIPS 2026 Workshop on Agents in the Wild).

A chain-of-thought monitor's pooled accuracy hides a subset of reward hacks it can only catch from
the agent's reasoning. Rewriting only that reasoning, with every command and output byte-identical,
drops the monitor's catch rate on the subset from about 95% to 4%, while the pooled rate barely moves.

## Layout

```
reproduce/   recompute every number in the paper from data/ (numpy only, no API key)
data/        the scores behind every table and figure (see data/README.md)
generate/    the pipelines that produced data/ (OpenRouter, the Terminal Wrench clone, GPU pods)
monitoring/  the Terminal Wrench authors' monitor (Apache-2.0), unmodified
prompts/     the Terminal Wrench hack prompt (Apache-2.0), given to the live agents
```

## Reproduce the paper

```bash
pip install numpy
python reproduce/all.py
```

It prints each number the paper reports beside the number the data gives, and exits non-zero if any
disagree. Each section also runs on its own:

| Script | Paper |
|---|---|
| `reproduce/s3_setup.py` | Section 3, Appendix B: the pool, the subset, thresholds, realized and split-half false-positive rates |
| `reproduce/s4_attack.py` | Section 4, Tables 1, 3 and 4: the rewrite, its robustness, the subset sensitivity, the label split |
| `reproduce/s5_defenses.py` | Section 5, Table 2, Appendix E: the defenses and the rewrite's faithfulness |
| `reproduce/s5_live.py` | Section 5, Table 5, Appendices H and J: live agents, the hidden channel, the rewrite on live hacks |
| `reproduce/s6_probe.py` | Section 6, Table 6, Appendix I: the probe and its behavioral controls |
| `reproduce/s7_distill.py` | Section 7, Appendix K: the LoRA distillation |

## Regenerate the data

```bash
pip install -r requirements.txt
git clone https://github.com/few-sh/terminal-wrench repo     # or set TW_REPO to an existing clone
export OPENROUTER_API_KEY=...                                 # RUNPOD_API_KEY and HF_TOKEN for GPU pods
python -m generate.attack.random_sample --selfcheck           # checks the rewrite rail
```

Run every generator from the repository root as a module: `data/<stage>/<name>` comes from
`python -m generate.<stage>.<name>`, and each script's docstring gives its inputs. Shared code lives in
`generate/common/`: the dataset interface (`tw.py`), the judge rubrics (`judge.py`), the attack
(`obfuscator.py`), the samples every experiment draws (`subset.py`), and one OpenRouter client
(`openrouter.py`). The exceptions to the naming rule:

| Data | Command (`python -m ...`) |
|---|---|
| `frozen_pool/*`, `opus_pool/*` | `generate.frozen_pool.score_pool` (`--help` lists the runs) |
| `attack/random_sample` | `generate.attack.random_sample --batch` |
| `defenses/legit_*` | written with their defense; `legit_skeptic` by `generate.attack.headline` |
| `live/<agent>` | `generate.live.launch --agent <model>` |
| `probe/probe_8b` | `generate.probe.prepare_inputs`, `launch_pod_8b`, `controls_8b` |
| `probe/probe_8b_action_only` | `generate.probe.action_only prepare`, `launch_pod_8b --inputs runs/probe/internals_inputs_ao.jsonl --no-verbal --tag ao`, `action_only collect` |
| `probe/probe_32b` | `generate.probe.prepare_inputs_32b`, `launch_pod_32b --part A` and `B`, `analyze_32b` |
| `distill/lora` | `generate.distill.prepare`, `launch_pod`, `lora` |

Pre-registrations: `generate/live/prereg_rewrite.md` and `generate/attack/prereg_checks.md`.

## License

MIT (see `LICENSE`). `monitoring/` and `prompts/` come from Terminal Wrench under Apache-2.0
(`monitoring/LICENSE-terminal-wrench`).
