# Jupyter Conda Environment (Minimal & Exportable)

This environment re-creates my former `pyenv` Jupyter setup using Conda, with only the top-level packages I installed. All other dependencies are resolved automatically by Conda.

---

## Environment File
`environment.yml` contains:
```yaml
name: jupyter-env
channels:
  - conda-forge
channel_priority: strict
dependencies:
  - python=3.12.3
  - pip
  - jupyterlab
  - ipykernel
  - ipywidgets
  - numpy
  - pandas
  - scipy
  - matplotlib
  - pillow
  - mendeleev
  - sqlalchemy
  - autograd
  - tensorly
```

---

## Create the Environment
```bash
conda env create -f environment.yml
conda activate jupyter-env
```

---

## Register the Kernel for VS Code (optional)
To make this environment available in VS Code Jupyter notebooks:
```bash
python -m ipykernel install --user --name jupyter-env --display-name "Python (jupyter-env)"
```

In VS Code, select **Python (jupyter-env)** as the kernel for your notebook.

---

## Launch Jupyter from VS Code
Simply open your notebook in VS Code and select the kernel above. No need to run JupyterLab in the browser.

---

## Export for Portability
To save a clean spec (only human-chosen packages):
```bash
conda env export --from-history > env/conda-from-history.yml
```

Recreate later:
```bash
conda env create -f env/conda-from-history.yml
conda activate jupyter-env
```

and optionally:
```bash
python -m ipykernel install --user --name jupyter-env --display-name "Python (jupyter-env)"
```

---

## Adding Packages
Prefer Conda (conda-forge):
```bash
conda install -c conda-forge <package>
```

Use pip only when a package isn’t available on conda-forge:
```bash
pip install <package>
```

---

## Notes
- Pinning `python=3.12.3` ensures compatibility with your previous pyenv setup.
- JupyterLab automatically installs its dependencies (e.g., `jupyter_core`, `jupyter_client`, `tornado`, `traitlets`, etc.).
- Matplotlib and pandas pull their own sub-dependencies (e.g., `contourpy`, `python-dateutil`, `pytz`).
- Keep `channels: [conda-forge]` and `channel_priority: strict` for consistent builds.
