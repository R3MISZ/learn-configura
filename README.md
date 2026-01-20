<p align="center"><img src="docs/assets/configura_logo.png" alt="Configura Logo" width="400"/></p>
<p align="center"><b>A lightweight, config-driven pipeline engine for structured data processing.</b></p>

---
### Why Configura

- Removes boilerplate and keeps data pipelines clean
- Pipelines are built from small, reusable components
- **Adapters** handle I/O, **plugins** handle logic
- Pipelines are defined entirely in **YAML**

---
### How Configura Works

Configura executes data pipelines defined in YAML.

Each **pipeline** consists of ordered steps.  
Every step is a Python class with a `process(data)` method.
`Data` is a variable passed through the pipeline as input and output for each step.

```powershell
For each step defined in YAML:
  data ──> [ClassPath:ClassName].process(data) ──> data
```

---
### Example Pipeline

YAML config:
```yaml
pipeline:
  - type: "configura.adapters.jsonl_adapter:ReadJsonl"
    params: { path: "data/input/records.jsonl" }

  - type: "configura.plugins.filter_by_field:FilterByField"
    params: { key_name: "value", operator: ">=", value: 20 }

  - type: "configura.adapters.jsonl_adapter:WriteJsonl"
    params: { path: "data/output/records_output.jsonl" }
```

Execution order:
```text
ReadJsonl → FilterByField → WriteJsonl
```

Execution flow:
```powershell
data = None
↓
ReadJsonl.process(data)
↓
data = [{"id": 1, "value": 10}, {"id": 2, "value": 20}, {"id": 3, "value": 25}, {"id": 4, "value": 5}]
↓
FilterByField.process(data)
↓
data = [{"id": 2, "value": 20}, {"id": 3, "value": 25}]
↓
WriteJsonl.process(data) # new file in "data/output/records_output.jsonl"
↓
data = None
```

---
## Project Structure

```bash
src/
  configura/
    cli.py                # CLI entrypoint
    engine.py             # Pipeline executor
    loader.py             # Dynamic class loader (adapters, plugins, ...)
    io.py                 # Input/output utilities

    adapters/             # Input/output components
      base_adapter.py
      json_adapter.py
      ...

    plugins/              # Data transformations
      filter_by_field.py
      ...
data/
  configs/                # Pipeline definitions
  input/                  # Raw data
  dlq/                    # Failed/invalid data
  output/                 # Processed data
```

---
## Quickstart

### 1. Installation
```bash
git clone https://github.com/R3MISZ/configura
cd configura
pip install -e .
```

### 2. Input data `data/input/records.jsonl`

```json
{"id": 1, "value": 10}
{"id": 2, "value": 20}
{"id": 3, "value": 25}
{"id": 4, "value": 5}
```

### 3. Pipeline `data/configs/pipeline.yaml`

```yaml
pipeline:
  - type: "configura.adapters.jsonl_adapter:ReadJsonl"
    params: { path: "data/input/records.jsonl" }

  - type: "configura.plugins.filter_by_field:FilterByField"
    params: { key_name: "value", operator: ">=", value: 20 }

  - type: "configura.adapters.jsonl_adapter:WriteJsonl"
    params: { path: "data/output/records_output.jsonl" }
```

### 4. Run

```bash
# Default
configura --config data/configs/pipeline.yaml

# Default + debug output
configura --config data/configs/pipeline.yaml --verbose
```

or

```python
# configura/my_module.py
from configura.engine import run_pipeline_from_config

run_pipeline_from_config("data/configs/pipeline.yaml")
```

### 5. Result `data/output/records_output.jsonl`
```json
{"id": 2, "value": 20}
{"id": 3, "value": 25}
```