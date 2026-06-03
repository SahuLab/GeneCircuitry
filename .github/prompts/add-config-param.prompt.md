---
mode: agent
description: "Add a new configuration parameter to GeneCircuitry. Use when: adding a new tunable value, exposing a hardcoded number as a config constant, extending the config system with a new setting."
---

Add a new configuration parameter to GeneCircuitry: $PARAM_DESCRIPTION

## Steps

1. **Add the constant** to [genecircuitry/config.py](../genecircuitry/config.py) as a module-level constant with a one-line docstring comment:

```python
# <Description of what this controls>
MY_PARAM: <type> = <default_value>
```

Follow the existing category groupings (QC, Preprocessing, Plotting, etc.) — place the new param in the appropriate section.

2. **Add to `get_config()`** in the same file so it is included in the returned dict:

```python
"MY_PARAM": MY_PARAM,
```

3. **Add a test** in [tests/test_config.py](../tests/test_config.py):

```python
def test_get_config_includes_my_param():
    cfg = config.get_config()
    assert "MY_PARAM" in cfg
    assert isinstance(cfg["MY_PARAM"], <type>)
```

4. **Use the param** everywhere it is needed via `from genecircuitry import config` then `config.MY_PARAM`. Never hardcode the value inline.

5. **Function signatures** that expose this param should default to `None` and fall back to config:

```python
def my_function(adata, my_param=None):
    if my_param is None:
        my_param = config.MY_PARAM
```

Run `pixi run -e dev test` (or `pytest tests/test_config.py -v`) to verify.
