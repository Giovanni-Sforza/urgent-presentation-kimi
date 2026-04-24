# HEP Project: {project-name}

## Pipeline Status

```yaml
stage: idle          # idle | idea-discovery | implementation | calculation | review | paper
idea: ""             # Current idea title (one line)
contract: ""         # Path to research_contract.md
current_branch: ""   # Git branch for this idea
hep_type: "pheno"    # pheno | theory — controls calculation and review defaults
baseline: ""         # Baseline results for comparison (e.g., known cross-section)
calculation_status: idle  # idle | running | complete | failed
language: en         # en | zh — controls skill output language
active_tasks: []
next: ""             # Concrete next step
last_updated: ""     # YYYY-MM-DD HH:mm — auto-updated by skills
```

## Project Constraints

- {constraint 1}
- {constraint 2}

## Non-Goals

- {non-goal 1}

## Compute Environment

```yaml
type: local          # local | remote | cluster | cloud
cores: 8
memory: 32GB
math_kernel: /usr/local/bin/math  # Mathematica kernel (optional)
python_env: conda env "hep-research"  # Python environment
hep_tools:
  - python3 + sympy + numpy + scipy + matplotlib
  # - mathematica
  # - madgraph5
  # - pythia8
  # - root
```

## Remote Server (if applicable)

- compute: remote
- SSH: `ssh my-hep-server`
- CPU: 32 cores, 128GB RAM
- Conda: `eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate hep-research`
- Code dir: `/home/user/hep-project/`
- code_sync: rsync

## Cluster (if applicable)

- compute: cluster
- SSH: `ssh my-cluster`
- Shared FS: `/lustre/user/project/`
- Default partition: `cpu`
- Account: `hep-group`

## Calculation Budget

- Max CPU-hours per calculation: 100
- Max total CPU-hours for project: 500
- Max wall time per job: 48 hours
