---
name: toksearch-basics
description: Basic TokSearch API usage - Pipeline, Record, and core data processing concepts for fusion experimental data
---

# TokSearch Basics Skill

## Description
TokSearch is a Python package for parallel retrieving, processing, and filtering of arbitrary-dimension fusion experimental data from tokamak experiments (primarily DIII-D). This skill covers the core API: Pipeline, Record, and basic data processing operations.

## When to Use
- When the user needs to analyze fusion experimental data from tokamaks
- When working with shot-based data processing workflows
- When fetching, filtering, and transforming experimental signals
- When performing parallel data analysis across multiple shots

## Core Concepts

### Pipeline
The `Pipeline` class is the main interface for data processing workflows. It processes `Record` objects (each representing data from a single experimental "shot").

```python
from toksearch import Pipeline

# Create a pipeline for a list of shot numbers
shots = [1701, 1702, 1703, 1704, 1705]
pipeline = Pipeline(shots)
```

### Record
Records are dict-like objects that hold data for a single shot. Each record has a 'shot' field and can contain multiple signals with their data.

```python
# Access data in a record (within map/where functions)
data = rec['signal_name']

# Get shot number (attribute access)
shot_number = rec.shot

# Check for errors
if 'signal_name' in rec.errors:
    # Handle error
    pass

# Get value with default (IMPORTANT: get() requires BOTH key and default)
value = rec.get('signal_name', None)
```

## Pipeline Methods

### fetch(signal_name, signal)
Fetch data from a signal source and add it to each record.

```python
from toksearch import MdsSignal

# Decorator style
@pipeline.fetch('electron_temperature', MdsSignal(r'\t_e', 'efit01'))
def my_fetch():
    pass

# Direct call style
pipeline.fetch('electron_temperature', MdsSignal(r'\t_e', 'efit01'))
```

### map(func)
Apply a function to each record, modifying it in place.

```python
@pipeline.map
def add_calculated_field(rec):
    rec['t_e_squared'] = rec['electron_temperature'] ** 2
```

### where(func)
Filter records based on a boolean condition.

```python
@pipeline.where
def has_valid_data(rec):
    return rec.get('electron_temperature', None) is not None and len(rec['electron_temperature']) > 0
```

### keep(fields) / discard(fields)
Keep or discard specific fields in each record. IMPORTANT: Takes a LIST of field names.

```python
pipeline.keep(['shot', 'electron_temperature', 'ip'])
pipeline.discard(['temp_data', 'intermediate_results'])
```

### compute()
Execute the pipeline and return results.

```python
records = pipeline.compute()
```

## Complete Example

```python
from toksearch import Pipeline, MdsSignal

# Create pipeline with shot numbers
shots = list(range(1701, 1710))
pipeline = Pipeline(shots)

# Add data fetching
@pipeline.fetch('ip', MdsSignal(r'\ipmhd', 'efit01'))
def fetch_ip():
    pass

# Add a transformation
@pipeline.map
def normalize_ip(rec):
    if 'ip' in rec and rec['ip'] is not None:
        signal_data = rec['ip']
        data = signal_data['data']
        times = signal_data['times']
        rec['ip_ma'] = float(np.max(data))  # Get max and convert to MA

# Add filtering
@pipeline.where
def ip_threshold(rec):
    return rec.get('ip_ma', 0) > 0.5

# Execute
results = pipeline.compute_serial()

# Access results
for rec in results:
    print(f"Shot {rec['shot']}: Ip = {rec.get('ip_ma', 0):.2f} MA")
```

## Computing Backends

TokSearch supports multiple execution backends:

```python
# Serial execution (default, for local testing)
results = pipeline.compute_serial()

# Multiprocessing (local parallel)
results = pipeline.compute_multiprocessing()

# Ray (distributed computing)
results = pipeline.compute_ray()

# Spark (large-scale distributed)
results = pipeline.compute_spark()
```

## Data Access Patterns

### Check if data exists before using
```python
@pipeline.map
def safe_processing(rec):
    if 'signal_name' in rec and rec['signal_name'] is not None:
        # Process the data
        rec['processed'] = rec['signal_name'] * 2
    else:
        rec['error'] = 'Signal not available'
```

### Handle errors gracefully
```python
@pipeline.map
def robust_processing(rec):
    try:
        rec['result'] = complex_calculation(rec['data'])
    except Exception as e:
        rec['calculation_error'] = str(e)
```

### Work with time series data
```python
@pipeline.map
def time_stats(rec):
    if 'timeseries' in rec:
        import numpy as np
        data = rec['timeseries']
        rec['mean'] = np.mean(data)
        rec['std'] = np.std(data)
        rec['max'] = np.max(data)
        rec['min'] = np.min(data)
```

## Shot Number Format
Shot numbers are integers representing specific experimental discharges:
- DIII-D shots: typically 4-5 digit numbers (e.g., 1701, 17453, 183504)
- Used as unique identifiers for each experiment

## Common Workflow Pattern

```python
from toksearch import Pipeline, MdsSignal

# 1. Define shots
shots = [1701, 1702, 1703, ...]

# 2. Create pipeline
pipeline = Pipeline(shots)

# 3. Fetch data
@pipeline.fetch('signal1', MdsSignal('\\tree::signal1', shot=record['shot']))
def fetch1():
    pass

# 4. Transform data
@pipeline.map
def transform(rec):
    rec['new_field'] = process(rec['signal1'])

# 5. Filter results
@pipeline.where
def filter(rec):
    return condition(rec)

# 6. Compute
results = pipeline.compute_serial()
```

## Best Practices
- Always check if data exists before accessing it
- Use descriptive signal names
- Handle errors gracefully with try/except blocks
- Test with small shot ranges first before scaling up
- Use `compute_serial()` for development, switch to parallel backends for production

## Notes
- Shot numbers are specific to each tokamak device
- Signal paths use MDSplus syntax: `\tree::signal_name`
- Records accumulate all operations in the pipeline
- Failed signal fetches are tracked as errors in the record
