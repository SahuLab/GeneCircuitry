---
mode: agent
description: "Add a new step to the GeneCircuitry pipeline. Use when: implementing a new analysis step, adding a new run_step_* method, integrating a new processing module into PipelineController."
---

Add a new pipeline step to the GeneCircuitry pipeline for: $STEP_DESCRIPTION

## Requirements

1. **Add a `run_step_<name>` method** to `PipelineController` in [genecircuitry/pipeline/controller.py](../genecircuitry/pipeline/controller.py) following this pattern:

```python
def run_step_<name>(self, adata, log_dir=None):
    """<docstring>"""
    from genecircuitry.pipeline import log_step
    from genecircuitry.logging_utils import log_error
    log_step("Controller.<Name>", "STARTED")
    try:
        input_hash = compute_input_hash(self.input_path)
        if check_checkpoint(log_dir or self.log_dir, "<name>", input_hash):
            log_step("Controller.<Name>", "SKIPPED (checkpoint)")
            return <cached_result>
        result = <processing_function>(adata)
        write_checkpoint(log_dir or self.log_dir, "<name>", input_hash)
        log_step("Controller.<Name>", "COMPLETED")
        return result
    except Exception as e:
        log_error("Controller.<Name>", e)
        raise
```

2. **Add the step name** to the `STEP_NAMES` list / step registry in `controller.py` so it is reachable via `--steps <name>` on the CLI.

3. **Any new config parameters** must be added to [genecircuitry/config.py](../genecircuitry/config.py) as module-level constants and included in the `get_config()` return dict, then tested in [tests/test_config.py](../tests/test_config.py).

4. **New plots** go into the appropriate module under [genecircuitry/plotting/](../genecircuitry/plotting/) and exported from `plotting/__init__.py`.

5. **New report sections** go into [genecircuitry/reporting/sections.py](../genecircuitry/reporting/sections.py) and are exported from `reporting/__init__.py`.

6. **Logging inside the new processing module** (not controller): use `from genecircuitry.logging_utils import log_error, log_warning` only.

7. **Optional deps**: if the step requires a non-core library, wrap the import in `try/except ImportError` in [genecircuitry/**init**.py](../genecircuitry/__init__.py) and set the module to `None` on failure.

Never create standalone scripts — always integrate into `PipelineController`.
