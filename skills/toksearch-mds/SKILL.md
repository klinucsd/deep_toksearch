---
name: toksearch-mds
description: MDSplus-specific data access in TokSearch - tree structures, signal paths, and DIII-D data retrieval
license: Apache-2.0
compatibility: Designed for deepagents CLI
metadata:
  author: GA-FDP
  version: "1.0"
  url: https://ga-fdp.github.io/toksearch/
---

# TokSearch MDSplus Skill

## Description
MDSplus (Multi-Dimensional Scale Plus) is the primary data storage system for DIII-D and many tokamaks. This skill covers MDSplus-specific data access patterns, tree structures, and common DIII-D signal retrieval.

## When to Use
- When fetching data from DIII-D MDSplus trees
- When working with EFIT equilibrium data
- When accessing PCS (Plasma Control System) data
- When retrieving magnetics, diagnostics data
- When you need time-series data from fusion experiments

## MDSplus Basics

### Tree Structure
MDSplus organizes data in hierarchical trees:

```
\tree::node::subnode::signal_name
```

**Common DIII-D Trees:**
- `efit` - EFIT equilibrium reconstruction
- `pcs` - Plasma Control System
- `magnetics` - Magnetic diagnostics
- `ece` - Electron Cyclotron Emission
- `thomson` - Thomson scattering
- `d3d` - General DIII-D data

### Signal Path Syntax

```python
from toksearch import MdsSignal

# CORRECT: MdsSignal takes expression and treename as separate arguments
signal = MdsSignal(r'\signal_name', 'treename')

# Examples
signal = MdsSignal(r'\ipmhd', 'efit01')      # Plasma current from EFIT
signal = MdsSignal(r'\ip', 'magnetics')     # Plasma current from magnetics
signal = MdsSignal(r'\ne', 'pcs')           # Density from PCS

# IMPORTANT:
# - Use raw strings r'...' for backslashes
# - Expression and treename are SEPARATE arguments
# - Do NOT use \tree::signal combined syntax
# - Do NOT pass shot parameter - Pipeline handles it
```

## Common DIII-D Signals by Category

### EFIT (Equilibrium) Signals

| Expression | Tree | Description | Units |
|------------|------|-------------|-------|
| `\ipmhd` | efit01 | Plasma current | MA |
| `\t_e` | efit01 | Electron temperature profile | keV |
| `\ne` | efit01 | Electron density profile | 10^20 m^-3 |
| `\betap` | efit01 | Poloidal beta | - |
| `\qmin` | efit01 | Minimum safety factor | - |
| `\q95` | efit01 | Safety factor at 95% flux | - |
| `\rmajor` | efit01 | Major radius | m |
| `\a` | efit01 | Minor radius | m |
| `\kappa` | efit01 | Elongation | - |
| `\delta` | efit01 | Triangularity | - |

### PCS (Plasma Control) Signals

| Expression | Tree | Description | Units |
|------------|------|-------------|-------|
| `\ts_ip` | pcs | Plasma current (PCS) | A |
| `\ne` | pcs | Line-averaged density | 10^20 m^-3 |
| `\gasb` | pcs | Gas injection rate | Torr-L/s |

### Magnetics Signals

| Expression | Tree | Description | Units |
|------------|------|-------------|-------|
| `\ip` | magnetics | Plasma current | A |
| `\vloop` | magnetics | Loop voltage | V |

### Diagnostic Signals

| Expression | Tree | Description | Units |
|------------|------|-------------|-------|
| `\t_e` | ece | Electron temperature (ECE) | eV |
| `\ne` | thomson | Electron density (Thomson) | 10^20 m^-3 |
| `\te` | thomson | Electron temperature (Thomson) | eV |

## Complete Examples

### Example 1: Basic EFIT Data

```python
from toksearch import Pipeline, MdsSignal

shots = [1701, 1702, 1703]
pipeline = Pipeline(shots)

# Fetch plasma current
@pipeline.fetch('ip_ma', MdsSignal(r'\ipmhd', 'efit01'))
def fetch_ip():
    pass

# Fetch q_min
@pipeline.fetch('qmin', MdsSignal(r'\qmin', 'efit01'))
def fetch_qmin():
    pass

# Filter for high performance
@pipeline.where
def high_qmin(rec):
    return rec.get('qmin', 0) > 2.0

results = pipeline.compute_serial()
```

### Example 2: Profile Data Analysis

```python
from toksearch import Pipeline, MdsSignal
import numpy as np

shots = list(range(1701, 1710))
pipeline = Pipeline(shots)

# Fetch temperature profile (returns rho grid + Te data)
@pipeline.fetch('te_profile', MdsSignal('\\efit::t_e', shot=record['shot']))
def fetch_te():
    pass

# Analyze profile
@pipeline.map
def analyze_profile(rec):
    if 'te_profile' in rec and rec['te_profile'] is not None:
        rho, te = rec['te_profile']

        # Calculate central temperature
        rec['te_central'] = te[np.argmin(np.abs(rho))]

        # Calculate gradient at mid-radius
        mid_idx = np.argmin(np.abs(rho - 0.5))
        if mid_idx < len(rho) - 1:
            gradient = (te[mid_idx+1] - te[mid_idx]) / (rho[mid_idx+1] - rho[mid_idx])
            rec['te_gradient'] = gradient

results = pipeline.compute_serial()
```

### Example 3: Multi-Tree Data

```python
from toksearch import Pipeline, MdsSignal

shots = [1701, 1702, 1703]
pipeline = Pipeline(shots)

# EFIT data
@pipeline.fetch('efit_ip', MdsSignal('\\efit::ipmhd', shot=record['shot']))
def fetch_efit_ip():
    pass

# PCS data
@pipeline.fetch('pcs_ip', MdsSignal('\\pcs::ts_ip', shot=record['shot']))
def fetch_pcs_ip():
    pass

# Compare sources
@pipeline.map
def compare_sources(rec):
    if 'efit_ip' in rec and 'pcs_ip' in rec:
        rec['ip_diff'] = rec['efit_ip'] - rec['pcs_ip']
        rec['ip_ratio'] = rec['efit_ip'] / rec['pcs_ip'] if rec['pcs_ip'] != 0 else None

results = pipeline.compute_serial()
```

## Time-Based Operations

### Working with Time Series

Most MDSplus signals return time-base + data arrays:

```python
@pipeline.fetch('ip_timeseries', MdsSignal('\\efit::ipmhd', shot=record['shot']))
def fetch_ip():
    pass

@pipeline.map
def process_timeseries(rec):
    if 'ip_timeseries' in rec:
        times, ip_data = rec['ip_timeseries']

        # Get statistics
        rec['ip_mean'] = np.mean(ip_data)
        rec['ip_max'] = np.max(ip_data)
        rec['ip_min'] = np.min(ip_data)

        # Find flat-top period
        ip_max = np.max(ip_data)
        flat_top_mask = ip_data > (0.95 * ip_max)
        rec['flattop_duration'] = np.sum(times[flat_top_mask])
```

### Time Window Selection

```python
@pipeline.map
def extract_time_window(rec):
    if 'ip_timeseries' in rec:
        signal_data = rec['ip_timeseries']
        times = signal_data['times']
        data = signal_data['data']

        # Extract data between 2.0 and 3.0 seconds
        mask = (times >= 2.0) & (times <= 3.0)
        rec['window_times'] = times[mask]
        rec['window_data'] = data[mask]
```

### Time-Based Alignment

```python
from toksearch import XarrayAligner

# Align multiple signals to common time grid
pipeline = Pipeline(shots)

@pipeline.fetch('te', MdsSignal('\\ece::t_e', shot=record['shot']))
def fetch_te():
    pass

@pipeline.fetch('ne', MdsSignal('\\thomson::ne', shot=record['shot']))
def fetch_ne():
    pass

# Align to common time grid
@pipeline.align('time', [2.0, 2.1, 2.2, 2.3, 2.4, 2.5])
def align_signals(rec):
    pass
```

## Error Handling

### Check Signal Availability

```python
@pipeline.map
def check_availability(rec):
    # Check individual signals
    if rec.has_error('te_profile'):
        rec['te_status'] = 'missing'
    elif rec['te_profile'] is None:
        rec['te_status'] = 'null'
    else:
        rec['te_status'] = 'ok'
```

### Validate Data Quality

```python
@pipeline.map
def validate_data(rec):
    if 'ip_timeseries' in rec:
        signal_data = rec['ip_timeseries']
        times = signal_data['times']
        data = signal_data['data']

        # Check for issues
        if len(times) == 0 or len(data) == 0:
            rec['data_quality'] = 'empty'
        elif np.any(np.isnan(data)):
            rec['data_quality'] = 'has_nan'
        elif np.any(np.isinf(data)):
            rec['data_quality'] = 'has_inf'
        else:
            rec['data_quality'] = 'good'
```

### Graceful Degradation

```python
@pipeline.map
def robust_calculation(rec):
    # Try primary signal
    if 'efit_ip' in rec and rec['efit_ip'] is not None:
        rec['ip'] = rec['efit_ip']
        rec['ip_source'] = 'efit'
    # Fall back to secondary
    elif 'pcs_ip' in rec and rec['pcs_ip'] is not None:
        rec['ip'] = rec['pcs_ip']
        rec['ip_source'] = 'pcs'
    # No data available
    else:
        rec['ip'] = None
        rec['ip_source'] = 'missing'
```

## Advanced MDSplus Operations

### Fetching Subtree/Node Information

```python
# Some nodes have multiple members
signal = MdsSignal('\\tree::node:member', shot=record['shot'])

# Example with units
signal = MdsSignal('\\tree::signal', shot=record['shot'],
                   units='A')  # Specify expected units
```

### Time-Based Slicing

```python
# Fetch specific time window
signal = MdsSignal('\\tree::signal', shot=record['shot'],
                   time_start=2.0,  # Start time
                   time_end=3.0)    # End time
```

### Handling Different Data Types

```python
# Scalar values
scalar_signal = MdsSignal('\\tree::scalar', shot=record['shot'])

# 1D arrays (time series)
timeseries_signal = MdsSignal('\\tree::timeseries', shot=record['shot'])

# 2D arrays (profiles)
profile_signal = MdsSignal('\\tree::profile', shot=record['shot'])
```

## Common Workflows

### Workflow 1: Shot Screening

```python
from toksearch import Pipeline, MdsSignal

shots = list(range(1701, 1800))
pipeline = Pipeline(shots)

@pipeline.fetch('ip', MdsSignal(r'\ipmhd', 'efit01'))
def fetch_ip():
    pass

@pipeline.fetch('qmin', MdsSignal(r'\qmin', 'efit01'))
def fetch_qmin():
    pass

# Screen for high-performance shots
@pipeline.where
def high_performance(rec):
    return (rec.get('ip', 0) > 1.5 and
            rec.get('qmin', 0) > 2.0)

results = pipeline.compute_serial()
high_perf_shots = [rec['shot'] for rec in results]
print(f"Found {len(high_perf_shots)} high-performance shots")
```

### Workflow 2: Time-Averaged Statistics

```python
from toksearch import Pipeline, MdsSignal
import numpy as np

shots = [1701, 1702, 1703]
pipeline = Pipeline(shots)

@pipeline.fetch('ip', MdsSignal('\\efit::ipmhd', shot=record['shot']))
def fetch_ip():
    pass

@pipeline.map
def time_averaged_stats(rec):
    if 'ip' in rec and rec['ip'] is not None:
        times, data = rec['ip']

        # Flat-top period (when Ip > 95% of max)
        max_ip = np.max(data)
        flattop_mask = data > (0.95 * max_ip)

        if np.any(flattop_mask):
            rec['flattop_ip_mean'] = np.mean(data[flattop_mask])
            rec['flattop_ip_std'] = np.std(data[flattop_mask])

results = pipeline.compute_serial()
```

## Best Practices
- Use specific tree names when possible (e.g., `\efit::` not `\d3d::`)
- Always check for errors with `rec.has_error()`
- Validate data before processing
- Consider time windows for long-pulse discharges
- Use shot number ranges, not individual shots for batch processing
- Test with a few shots before scaling to thousands

## Notes
- MDSplus requires active server connection
- Signal paths are case-sensitive
- Not all shots have all signals
- Time bases vary between diagnostics
- Use `compute_serial()` for development
