# jupyter-env

A portable Python/Jupyter environment using Conda. Designed to work in two
contexts: as a **submodule inside a devcontainer**, where installation is
fully automatic, and as a **standalone local install** on any machine with
Conda available.

---

## Devcontainer usage

If you are opening a project that includes this repo as a submodule at
`.devcontainer/jupyter-env/`, nothing is required. The devcontainer's
`install.sh` handles everything automatically:

1. Creates the `jupyter-env` conda environment from `environment.yml`
2. Registers the IPython kernel so VS Code and JupyterLab both see it

Just open the project in VS Code and click **Reopen in Container**.

---

## Local install

### 1. Create and activate the conda environment

```bash
conda env create -f environment.yml
conda activate jupyter-env
```

### 2. Register the kernel

```bash
python -m ipykernel install --user --name jupyter-env --display-name "Python (jupyter-env)"
```

In VS Code, select **Python (jupyter-env)** as the kernel for your notebook.

### 3. Verify your setup

```bash
# Check Python and key packages
python --version
python -c "import numpy, pandas, scipy, matplotlib, pyscf; print('all good')"

# Check JupyterLab
jupyter lab --version

# Check kernel is registered
jupyter kernelspec list
```

### Uninstall

```bash
conda deactivate
conda remove --name jupyter-env --all
jupyter kernelspec remove jupyter-env
```

---

## Updating packages

### Adding or removing packages

Always edit `environment.yml` first, then apply the change to the live env.
Never install ad-hoc with `conda install` without also updating the yml —
that is how the two fall out of sync.

```bash
# 1. Edit environment.yml (add/remove/change a package)
# 2. Apply to the live environment
conda env update -n jupyter-env -f environment.yml --prune

# 3. Commit the change
git add environment.yml
git commit -m "add <package>"
git push
```

The `--prune` flag ensures packages removed from the yml are also removed
from the live env, keeping the two in sync.

After pushing, downstream projects that use this repo as a submodule should
bump their submodule pointer:

```bash
# Inside the project repo (not inside the submodule)
git submodule update --remote --init --recursive .devcontainer/jupyter-env
git add .devcontainer/jupyter-env
git commit -m "bump jupyter-env"
git push
```

Then rebuild the devcontainer in VS Code: right-click the remote indicator
(bottom-left) → **Rebuild Container**.

---

## Devcontainer integration

This repo is intended to be used as a git submodule inside a Jupyter project's
`.devcontainer/` folder. The expected project-side layout is:

```text
.devcontainer/
├── devcontainer.json
├── install.sh
└── jupyter-env/          ← this repo as a submodule
```

### Adding to a project

```bash
git submodule add https://github.com/mariusmayer/jupyter-env.git .devcontainer/jupyter-env
git submodule update --init --recursive
```

### Minimal devcontainer.json

```json
{
    "name": "jupyter-env",
    "image": "mcr.microsoft.com/devcontainers/miniconda:latest",
    "initializeCommand": "git submodule update --init --recursive",
    "postCreateCommand": "bash .devcontainer/install.sh 2>&1 | tee .devcontainer/install.log",
    "customizations": {
        "vscode": {
            "extensions": [
                "ms-python.python",
                "ms-python.pylance",
                "ms-toolsai.jupyter",
                "mechatroner.rainbow-csv"
            ],
            "settings": {
                "python.defaultInterpreterPath": "/opt/conda/envs/jupyter-env/bin/python",
                "jupyter.kernels.filter": [
                    {
                        "path": "/opt/conda/envs/jupyter-env/bin/python",
                        "type": "pythonEnvironment"
                    }
                ]
            }
        }
    },
    "remoteUser": "vscode"
}
```

### Minimal install.sh

The project-side `install.sh` is a thin orchestrator that delegates entirely
to this submodule's `environment.yml`:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
ENV_DIR="$SCRIPT_DIR/jupyter-env"

source "$(conda info --base)/etc/profile.d/conda.sh"

if conda env list | grep -q "^jupyter-env "; then
    conda env update -n jupyter-env -f "$ENV_DIR/environment.yml" --prune
else
    conda env create -f "$ENV_DIR/environment.yml"
fi

conda run -n jupyter-env python -m ipykernel install \
    --user \
    --name jupyter-env \
    --display-name "Python (jupyter-env)"
```

---

## Repository contents

| File | Purpose |
|---|---|
| `environment.yml` | Conda environment spec (single source of truth) |
| `install.sh` | Standalone local helper (create env + register kernel) |

---

## Troubleshooting

**Kernel not showing in VS Code**: make sure the kernel was registered and
reload the VS Code window:

```bash
jupyter kernelspec list
# if jupyter-env is missing:
conda run -n jupyter-env python -m ipykernel install --user --name jupyter-env --display-name "Python (jupyter-env)"
```

**Package conflicts on update**: conda-forge with `channel_priority: strict`
is usually self-consistent, but if a conflict arises, recreate from scratch:

```bash
conda deactivate
conda remove --name jupyter-env --all
conda env create -f environment.yml
```

**`conda env update` feels slow**: this is normal for large solves on
conda-forge. Consider installing `mamba` as a drop-in solver:

```bash
conda install -n base -c conda-forge mamba
mamba env update -n jupyter-env -f environment.yml --prune
```