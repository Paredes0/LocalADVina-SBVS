# Contributing to the Local SBVS Pipeline

Thanks for your interest in improving this project! It started as a tool to
make Structure-Based Virtual Screening more accessible, and contributions of
any size are welcome.

## Ways to contribute

- **Report a bug** — open an issue describing what you ran, what you expected,
  and what happened (include error messages and your OS / hardware).
- **Suggest a feature** — open an issue explaining the use case.
- **Improve the documentation** — clarifications to the README or notebook
  markdown cells are very valuable, especially around setup and path handling.
- **Submit code** — bug fixes or new notebook cells / scripts.

## Setting up the environment

The full toolchain is pinned in [`environment.yml`](environment.yml):

```bash
micromamba create -f environment.yml
micromamba activate docking_vina
```

For the Colab ↔ local-machine connection, create the lightweight
`colab_connect` environment as described in the
[README](README.md#4-link-colab-with-the-local-environment).

## Pull request guidelines

1. Fork the repository and create a topic branch.
2. Keep changes focused — one logical change per pull request.
3. If you edit `pipeline_SBVS.ipynb`, **clear all cell outputs** before
   committing (`Kernel → Restart & Clear Output`) to keep diffs readable and
   avoid committing local paths or result data.
4. Describe what you changed and why in the PR description.
5. Make sure the notebook still runs end-to-end with a small test set of
   ligands.

## Coding notes

- The notebook generates helper scripts (`prepare_library.py`, `run_sbvs.py`,
  `run_redocking.py`) at runtime; these are git-ignored. Edit the generating
  cell, not the generated file.
- Prefer `subprocess` argument lists over shell strings for new docking calls.
- Keep user-facing prompts and messages consistent in language within a cell.

## Code of conduct

Be respectful and constructive. We want this to be a welcoming project for
students and researchers regardless of experience level.
