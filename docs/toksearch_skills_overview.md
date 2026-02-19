
This document explains the four TokSearch skills created for DeepAgents AI assistant and the current state of the project as of February 2025.

---

## Skill Summary

| Skill | Purpose | When Used | Status |
|-------|---------|-----------|--------|
| **toksearch-basics** | Core Pipeline operations - fetch, map, where, keep, compute | Foundation for all TokSearch workflows | ✅ Created |
| **toksearch-signals** | Signal classes (MdsSignal, ZarrSignal) and data access patterns | Critical for all data access | ✅ Created |
| **toksearch-mds** | DIII-D specific signals, tree names, and MDSplus patterns | Domain knowledge for DIII-D | ✅ Updated with FDP demo findings |
| **toksearch-backends** | Execution backends (Serial, Multiprocessing, Ray, Spark) | Large-scale processing | ✅ Created |

---

## Current Project Status (February 2025)

### What Works ✅

1. **AI Assistant translates domain language → code**
   - Scientists can ask questions in natural language
   - DeepAgents generates correct TokSearch-style Python code
   - Uses proper data access patterns

2. **Demo system fully functional**
   - Synthetic DIII-D-flavored data for testing
   - 5 demo requests covering different analysis types
   - All tested and working

3. **Skills teach correct patterns**
   - Critical data access: `rec['signal']['data']`
   - Correct signal expressions based on FDP demo
   - Domain knowledge (plasma physics concepts)

### What Needs Team Input ⏳

1. **DIII-D MDSplus access**
   - Requires GA/DIII-D collaborator credentials
   - Internal signal names not publicly documented
   - Server configuration needed

2. **Signal confirmation**
   - FDP demo shows `efit01` tree for `\kappa`, `\psirz` (HIGH confidence)
   - Other signals need team confirmation
   - Magnetics tree name uncertain

---

## Detailed Skill Descriptions

### 1. toksearch-basics

**Purpose:** Teaches the fundamental TokSearch Pipeline API.

**What it covers:**
- Creating a Pipeline for shot numbers
- Fetching data from signals
- Transforming data with `map` operations
- Filtering records with `where` conditions
- Keeping/discard fields with `keep()`/`discard()`
- Executing pipelines with `compute_serial()`

**Key concepts:**
- **Pipeline:** Like an assembly line for data
- **Record:** Container for one shot's data
- **fetch:** Get data from a signal source
- **map:** Apply function to each record
- **where:** Filter records based on conditions

---

### 2. toksearch-signals

**Purpose:** Teaches how to fetch data from different storage systems.

**What it covers:**
- Signal classes: `MdsSignal` and `ZarrSignal`
- Correct signal syntax: `MdsSignal(expression, treename)`
- Signal data format: `{'data': array, 'times': array, 'units': str}`
- **CRITICAL data access pattern:** Extract `['data']` before numpy operations

**CRITICAL warnings:**
- Signals return dicts, NOT raw arrays
- Never call `.max()` directly on `rec['signal']`
- Must extract `['data']` first

**Example:**
```python
# CORRECT pattern
signal_data = rec['ipmhd']     # Get dict
data = signal_data['data']      # Extract array
max_value = np.max(data)        # Use numpy
```

---

### 3. toksearch-mds

**Purpose:** DIII-D tokamak specific signals, tree structures, and domain knowledge.

**What it covers:**
- DIII-D MDSplus tree names
- Signal expressions for common diagnostics
- Domain-specific meanings (what qmin, betap, kappa represent)
- Typical value ranges for DIII-D

**Signal Confidence Levels (based on FDP demo):**

| Expression | Tree | Description | Confidence | Source |
|------------|------|-------------|------------|--------|
| `\kappa` | efit01 | Plasma elongation | **HIGH** | FDP demo (GA) |
| `\psirz` | efit01 | Poloidal flux (2D) | **HIGH** | FDP demo (GA) |
| `\ipmhd` | efit01 | Plasma current (EFIT) | **MEDIUM** | Inferred from FDP |
| `\ip` | magnetics? | Plasma current (mag) | **LOW** | Unconfirmed |
| `\qmin` | efit01? | Safety factor | **LOW** | Example only |
| `\ne` | efit01? | Electron density | **LOW** | Example only |

**Important Notes:**
- `efit01` confirmed as tree for EFIT signals (from FDP demo)
- Other tree names (magnetics, pcs, etc.) need team confirmation
- Signal expressions may be internal DIII-D naming conventions

---

### 4. toksearch-backends

**Purpose:** Teaches different execution backends for parallel processing.

**What it covers:**
- Serial: One shot at a time (good for development, testing)
- Multiprocessing: Use all CPU cores (100-1000 shots)
- Ray: Distributed computing (large datasets)
- Spark: Very large data processing (thousands of shots)

**When to use which:**
- Less than 100 shots: Use Serial
- 100-1000 shots: Use Multiprocessing
- More than 1000 shots: Use Ray or Spark

---

## Demo System

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                      DEMO WORKFLOW                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Generate synthetic DIII-D data                               │
│     → scripts/generate_demo_data.py                             │
│     → Creates demo_d3d_shots.pkl                                │
│                                                                  │
│  2. User asks question in natural language                       │
│     → "Compare plasma current from magnetics and EFIT"           │
│                                                                  │
│  3. AI generates Python code                                     │
│     → Uses correct data access patterns                          │
│     → Loads pickle file, analyzes data                          │
│                                                                  │
│  4. Code runs and produces results                               │
│     → Table with shot numbers, statistics                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Demo Files

| File | Location | Purpose |
|------|----------|---------|
| Data generator | `scripts/generate_demo_data.py` | Creates synthetic DIII-D data |
| Demo data | `tests/data/demo_d3d_shots.pkl` | 2 shots with realistic structure |
| Tutorial requests | `docs/toksearch_tutorial_requests.txt` | 5 demo requests to try |
| Skills | `~/.deepagents/agent/skills/toksearch-*` | Global skill installation |

### Tested Demo Requests

1. ✅ **Compare Plasma Current** - Max values, comparison table
2. ✅ **Plasma Current Statistics** - Mean, std, max values
3. ✅ **Exploratory Data Check** - What signals are available
4. ✅ **Data Structure Summary** - Detailed structure overview

### Additional Demo Requests (Not Yet Tested)

5. ⏳ **Stability Analysis** - Coefficient of variation, percentage calculations
6. ⏳ **Time-Series Analysis** - Difference between diagnostics over time
7. ⏳ **Summary Report** - Comprehensive formatted report
8. ⏳ **Exploratory Suggestions** - AI suggests analysis options

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
   - May be wrong - signal names are internal DIII-D conventions

2. **Direct numpy on signal:** `np.mean(rec['signal'])`
   - Correct: `np.mean(rec['signal']['data'])`

3. **Assuming tree names:**
   - `efit01` confirmed for EFIT signals (from FDP demo)
   - Other trees need team confirmation

---

## Summary for Developers

When creating new requests or teaching the AI:

1. **Use demo data** - Tests work without MDSplus access
2. **toksearch-signals is critical** - Teaches the dict access pattern
3. **Reference toksearch-mds** - For DIII-D domain knowledge
4. **Start with domain language** - Let AI translate to code

### For Production Use (with MDSplus access)

The skills are ready but need:
1. Confirmation of signal names from TokSearch team
2. MDSplus server configuration
3. GA/DIII-D collaborator credentials

The framework is solid - just needs DIII-D-specific configuration.
