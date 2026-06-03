---
applyTo: "genecircuitry/plotting/**"
description: "Conventions for adding or editing plots in the genecircuitry/plotting/ subpackage."
---

# Plotting Subpackage Conventions

See [docs/plotting.md](../../docs/plotting.md) for full documentation.

## File layout

New plots go into the module that matches the analysis area — never into `utils.py` or deprecated modules:

| Module | Scope |
|---|---|
| `qc_plots.py` | QC violin / scatter plots |
| `grn_plots.py` | Network, heatmap, TF centrality plots |
| `hotspot_plots.py` | Local-correlation, annotation, module-score plots |
| `utils.py` | Shared helpers only — `save_plot`, `plot_exists`, `PlotLogger` |

## Required function signature

Every public plot function must follow this signature:

```python
def plot_<name>(
    adata,                          # AnnData (or oracle / hs object)
    save_name: str = "default",
    figsize=None,                   # falls back to config.PLOT_FIGSIZE_*
    skip_existing: bool = True,
) -> bool:                          # True = generated, False = skipped
```

## Mandatory utilities — never skip

```python
from .. import config
from .utils import save_plot, plot_exists

filepath = f"{config.FIGURES_DIR_<AREA>}/<plot_name>_{save_name}.png"

if plot_exists(filepath, skip_existing):
    return False

fig, ax = plt.subplots(figsize=figsize or config.PLOT_FIGSIZE_MEDIUM)
# ... build figure ...

return save_plot(fig, filepath, dpi=config.SAVE_DPI)
```

- Path prefix: `config.FIGURES_DIR_QC`, `config.FIGURES_DIR_GRN`, or `config.FIGURES_DIR_HOTSPOT`
- Figure sizes: `config.PLOT_FIGSIZE_SMALL/MEDIUM/LARGE/WIDE/SQUARED` — never hardcode
- DPI: `config.SAVE_DPI` — never hardcode
- Never call `plt.savefig()` directly — always use `save_plot()`

## `generate_all_*` convention

Each module must expose one top-level aggregator that calls every individual plot function and returns `dict[str, bool]` (`{plot_name: was_generated}`):

```python
def generate_all_<area>_plots(adata, output_dir=None, skip_existing=True, **kwargs) -> dict:
    results = {}
    results["plot_a"] = plot_a(adata, skip_existing=skip_existing, **kwargs)
    results["plot_b"] = plot_b(adata, skip_existing=skip_existing, **kwargs)
    return results
```

## PlotLogger registration (preferred)

```python
from .utils import get_plot_logger

logger = get_plot_logger(output_dir)
logger.register_plot(filepath, plot_type="<area>", metadata={"step": "<name>"})
logger.save()
```

## Export rule

Every public symbol must appear in **both** the `from .<module> import ...` block **and** in `__all__` in [`genecircuitry/plotting/__init__.py`](../../genecircuitry/plotting/__init__.py).

## Anti-patterns

- Do not add plots inline in `grn_deep_analysis.py` or `hotspot_processing.py` — that code is deprecated
- Do not call `plt.savefig()` directly
- Do not hardcode figure sizes, DPI, or output paths
