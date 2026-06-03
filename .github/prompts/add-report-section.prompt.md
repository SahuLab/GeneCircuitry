---
mode: agent
description: "Add a new section to the GeneCircuitry analysis report. Use when: adding a report section, creating a new create_*_section function, wiring a new analysis step into the HTML/PDF report."
---

Add a new report section to GeneCircuitry for: $SECTION_DESCRIPTION

See [docs/reporting.md](../docs/reporting.md) for full reporting module documentation.

## Steps

### 1. Add `create_<name>_section()` to `reporting/sections.py`

Add the function to [genecircuitry/reporting/sections.py](../genecircuitry/reporting/sections.py) following this pattern:

```python
def create_<name>_section(
    adata=None,
    output_dir: str = None,
    # add result-specific kwargs here (e.g. celloracle_result=None)
) -> ReportSection:
    """
    Create <Name> section.
    """
    metrics = {}
    tables = []

    # Collect metrics
    # metrics["My Metric"] = some_value

    content = """
    <p>Description of this analysis step.</p>
    <h3>Sub-heading</h3>
    <ul>
        <li><strong>Key:</strong> Value</li>
    </ul>
    """

    # Discover figures using the helper already in sections.py
    figures = _find_figures(output_dir, ["figures/<area>"])

    return ReportSection(
        title="<Section Title>",
        section_id="<section-id>",
        content=content,
        figures=figures,
        metrics=metrics,
        tables=tables,
    )
```

- Use `_find_figures(output_dir, [...])` — already defined in `sections.py` — to collect plot paths
- `metrics` keys become summary cards in the report; keep values as plain Python scalars
- `tables` entries are `{"title": str, "headers": list[str], "rows": list[list]}`
- `section_id` must be a lowercase hyphen-separated string (used as the HTML anchor)

### 2. Export from `reporting/__init__.py`

In [genecircuitry/reporting/__init__.py](../genecircuitry/reporting/__init__.py), add the new function to **both** the `from .sections import ...` block and the `__all__` list.

### 3. Wire into `generate_report()` in `reporting/generator.py`

In [genecircuitry/reporting/generator.py](../genecircuitry/reporting/generator.py), locate the `generate_report()` function and add:

```python
from .sections import create_<name>_section

# Inside generate_report(), after the related existing sections:
generator.add_section(create_<name>_section(
    adata=adata,
    output_dir=output_dir,
    # pass result kwargs if needed
))
```

### 4. (Optional) Call from `PipelineController`

If the section corresponds to a pipeline step, add it inside the matching `run_step_*` method in [genecircuitry/pipeline/controller.py](../genecircuitry/pipeline/controller.py) so it is included in the per-run report.

## Checklist

- [ ] `create_<name>_section()` added to `sections.py` returning `ReportSection`
- [ ] Exported from `reporting/__init__.py` (import block + `__all__`)
- [ ] Wired into `generate_report()` in `generator.py`
- [ ] `section_id` is unique and slug-formatted
