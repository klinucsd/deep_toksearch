# TokSearch Skills Overview

This document explains the four TokSearch skills created for DeepAgents AI assistant and how they were used in the test requests.

---

## Skill Summary

| Skill | Purpose | When Used |
|-------|---------|-----------|
| **toksearch-basics** | Core Pipeline operations - fetch, map, where, keep, compute | All requests |
| **toksearch-signals** | Signal classes (MdsSignal, ZarrSignal) and data access patterns | All requests |
| **toksearch-mds** | DIII-D specific signals, tree names, and MDSplus patterns | Used for signal names |
| **toksearch-backends** | Execution backends (Serial, Multiprocessing, Ray, Spark) | Not used in test requests |

---

## Detailed Skill Descriptions

### 1. toksearch-basics

**Purpose:** Teaches the fundamental TokSearch Pipeline API - the main tool for data processing.

**What it covers:**
- Creating a Pipeline for shot numbers
- Fetching data from signals
- Transforming data with `map` operations
- Filtering records with `where` conditions
- Keeping/discard fields with `keep()`/`discard()`
- Executing pipelines with `compute_serial()`

**Key concepts:**
- **Pipeline:** Like an assembly line for data - shots go in, processed results come out
- **Record:** A container holding all data for one shot
- **fetch:** Get data from a signal source
- **map:** Apply a function to each record (transforms data)
- **where:** Filter records based on conditions (keeps or removes shots)
- **keep:** Keep only specific fields in the final results

**Example:**
```python
pipeline = Pipeline([1701, 1702, 1703])
pipeline.fetch('ip', MdsSignal(r'\ipmhd', 'efit01'))
results = pipeline.compute_serial()
```

---

### 2. toksearch-signals

**Purpose:** Teaches how to fetch data from different storage systems (MDSplus, Zarr).

**What it covers:**
- Signal classes: `MdsSignal` and `ZarrSignal`
- Correct signal syntax: `MdsSignal(expression, treename)` - NOT `MdsSignal('\tree::signal', shot=...)`
- Signal data format: Signals return dictionaries with `'data'`, `'times'`, `'units'` keys
- Critical data access pattern: MUST use `signal_data['data']` before calling numpy functions
- Common DIII-D signals and their correct names

**Critical warnings:**
- Magnetics tree uses `\ip`, NOT `\ipmeas`
- Signals return dicts, NOT raw arrays
- Never call `.max()` directly on `rec['signal']` - must extract `['data']` first

**Example:**
```python
# CORRECT pattern
signal_data = rec['ipmhd']     # Get dict
data = signal_data['data']      # Extract array
max_value = np.max(data)        # Use numpy
```

---

### 3. toksearch-mds

**Purpose:** DIII-D tokamak specific signals, tree structures, and common analysis patterns.

**What it covers:**
- Common DIII-D MDSplus tree names: `efit01`, `magnetics`, `pcs`, `ece`, `thomson`
- Signal expressions for frequently accessed diagnostics
- Typical analysis workflows for DIII-D data
- Domain-specific signal meanings (what qmin, betap, etc. represent physically)

**Common signals:**
| Expression | Tree | Description | Units |
|------------|------|-------------|-------|
| `\ipmhd` | efit01 | Plasma current | MA |
| `\t_e` | efit01 | Electron temperature profile | keV |
| `\ne` | efit01 | Electron density profile | 10^20 m^-3 |
| `\qmin` | efit01 | Minimum safety factor | - |
| `\ip` | magnetics | Plasma current | A |

---

### 4. toksearch-backends

**Purpose:** Teaches different execution backends for parallel processing.

**What it covers:**
- Serial: One shot at a time (good for development, testing, small datasets)
- Multiprocessing: Use all CPU cores on one computer (100-1000 shots)
- Ray: Distributed computing across multiple computers (large datasets)
- Spark: Very large data processing (thousands of shots)

**When to use which:**
- Less than 100 shots: Use Serial
- 100-1000 shots: Use Multiprocessing
- 1000-10000 shots: Use Ray or Multiprocessing
- More than 10000 shots: Use Ray or Spark

**Not used in test requests** because they involved only 2-6 shots.

---

## Skills Used in Test Requests

### Test Request 1: Basic Plasma Current Analysis

**Requirements:**
- Analyze shots 165920-165921
- Fetch ipmeas (magnetics) and ipmhd (efit)
- Calculate maximum values
- Filter where ipmhd > 1.0 MA
- Display results

**Skills used:**
- **toksearch-basics:** Pipeline, fetch, map, where, keep, compute_serial
- **toksearch-signals:** MdsSignal syntax, data access pattern (`['data']`)
- **toksearch-mds:** Signal names (`\ipmhd`, `\ip`), tree names (`efit01`, `magnetics`)

**Skills NOT used:**
- toksearch-backends (only 2 shots, serial is fine)

### Test Request 2: Extended Plasma Current Analysis with Statistics

**Requirements:**
- Analyze shots 165920-165925
- Fetch ipmhd from efit
- Calculate mean, std, max
- Display formatted results

**Skills used:**
- **toksearch-basics:** Pipeline, fetch, map, compute_serial
- **toksearch-signals:** MdsSignal syntax, critical data access pattern
- **toksearch-mds:** Signal name (`\ipmhd`), tree name (`efit01`)

**Skills NOT used:**
- toksearch-backends (only 6 shots, serial is fine)

---

## Key Learning Points

### Most Important Pattern

The **critical data access pattern** that must be followed:

```python
# Step 1: Get the signal dictionary
signal_data = rec['signal_name']

# Step 2: Extract the data array
data = signal_data['data']

# Step 3: Use numpy functions
value = np.max(data)
```

### Common Mistakes to Avoid

1. **Wrong signal expression:** `MdsSignal(r'\ipmeas', 'magnetics')`
   - Correct: `MdsSignal(r'\ip', 'magnetics')`

2. **Direct numpy on signal:** `np.mean(rec['signal'])`
   - Correct: `np.mean(rec['signal']['data'])`

3. **Wrong compute method:** `pipeline.compute()`
   - Correct: `pipeline.compute_serial()`

---

## Summary for Developers

When creating new requests or teaching the AI:

1. **Start with toksearch-basics** - All requests use Pipeline operations
2. **Use toksearch-signals** for any data fetching - the dict access pattern is critical
3. **Reference toksearch-mds** for DIII-D specific signal names
4. **Only use toksearch-backends** when processing large datasets (>100 shots)

The skills are designed to be composable - most requests will use basics + signals + domain-specific information from mds.
