# epMCP Tool Reference

This document describes the callable tools exposed by **epMCP**, the inputs they require, the outputs they generate, and example natural language queries that invoke them.

---

# Tool Overview

| Tool | Function | Required Inputs | Output |
|------|----------|-----------------|--------|
| Initialize Model | `init(idd_path, idf_path)` | `idd_path`, `idf_path` | Initialized EnergyPlus IDF object |
| Configure Simulation Period | `runperiod(idf_object, start_date, end_date)` | `idf_object`, `start_date`, `end_date` | Updated simulation period |
| Run EnergyPlus Simulation | `juzcsv(idf_object, epw_path, output_directory)` | `idf_object`, `epw_path`, `output_directory` | `eplusout.csv` and EnergyPlus outputs |
| Modify Global Occupancy | `altglo(idf_object, parameter_type, multiplier, save_directory, new_filename)` | `idf_object`, `"people"`, `multiplier`, `save_directory` | Modified IDF model |
| Extract Zones | `zones(idf_object)` | `idf_object` | Zone metadata |
| Extract Surfaces | `surfaces(idf_object)` | `idf_object` | Surface metadata |
| Extract Windows | `windows(idf_object)` | `idf_object` | Window metadata |
| Extract Materials | `materials(idf_object)` | `idf_object` | Material properties |
| Extract Occupancy | `occupancy(idf_object)` | `idf_object` | PEOPLE object information |
| Get Zone List | `zonearray(idf_object)` | `idf_object` | List of thermal zones |
| Extract Weather Data | `weather(epw_path)` | `epw_path` | Weather DataFrame |
| Baseline Visualization | `baseviz(idf_object, baseout_dir)` | `idf_object`, `baseout_dir` | Building analytics dashboard |
| Weekly Performance Analysis | `weeksim(baseout_dir)` | `baseout_dir` | Weekly load statistics and plots |
| Zone Temperature Heatmap | `heatmapzone(dataframe, zone_name)` | `eplusout.csv`, `zone_name` | 365×24 temperature matrix |
| Simulation Comparison | `hourlycomplot(...)` | Baseline CSV, Altered CSV | Comparative plots |

---

# Tool Input Requirements

Before executing any simulation-based workflow, the following files are required.

| Input | Description |
|--------|-------------|
| IDD | EnergyPlus Data Dictionary (`Energy+.idd`) |
| IDF | EnergyPlus building model |
| EPW | Weather file |
| Baseline Output Directory | Directory for baseline simulation outputs |
| Altered Output Directory | Directory for modified simulation outputs |

---

# Prompt → Tool Mapping

## 1. Building Summary

### Example Prompt

> Summarize the uploaded EnergyPlus building.

### Tool Sequence

```
init
    ↓
zones
    ↓
surfaces
    ↓
windows
    ↓
materials
    ↓
occupancy
```

### Expected Output

- Number of thermal zones
- Building floor area
- Window area
- Window-to-wall ratio
- Material properties
- Occupancy information

---

## 2. Run Baseline Simulation

### Example Prompt

> Run a baseline simulation for this building.

### Tool Sequence

```
init
    ↓
runperiod
    ↓
juzcsv
```

### Expected Output

- EnergyPlus simulation
- `eplusout.csv`

---

## 3. Generate Baseline Analytics

### Example Prompt

> Generate the baseline building analytics dashboard.

### Tool Sequence

```
init
    ↓
runperiod
    ↓
juzcsv
    ↓
baseviz
```

### Expected Output

- Building statistics
- Energy breakdown
- Load duration curve
- Material properties
- Occupancy metrics

---

## 4. Weekly Performance Analysis

### Example Prompt

> Show the typical, hottest and coldest operating weeks.

### Tool Sequence

```
init
    ↓
runperiod
    ↓
juzcsv
    ↓
weeksim
```

### Expected Output

- Typical week profile
- Peak demand week
- Minimum demand week
- Annual statistics

---

## 5. Zone Temperature Heatmap

### Example Prompt

> Generate the annual temperature heatmap for Zone-1.

### Tool Sequence

```
init
    ↓
runperiod
    ↓
juzcsv
    ↓
heatmapzone
```

### Expected Output

365 × 24 hourly temperature matrix

---

## 6. Modify Building Occupancy

### Example Prompt

> Increase occupancy by 50%.

### Tool Sequence

```
init
    ↓
altglo
```

### Expected Output

Modified IDF model

---

## 7. Modify Occupancy and Simulate

### Example Prompt

> Increase occupancy by 50% and run the simulation.

### Tool Sequence

```
init
    ↓
altglo
    ↓
runperiod
    ↓
juzcsv
```

### Expected Output

- Modified IDF
- Altered EnergyPlus simulation
- Updated `eplusout.csv`

---

## 8. Compare Baseline and Altered Simulations

### Example Prompt

> Compare the baseline model with a model having 50% higher occupancy.

### Tool Sequence

```
Baseline

init
    ↓
runperiod
    ↓
juzcsv

────────────

Modified

altglo
    ↓
runperiod
    ↓
juzcsv

────────────

hourlycomplot
```

### Expected Output

- Baseline vs altered demand plots
- Hourly comparison
- Peak demand comparison

---

## 9. Extract Weather Information

### Example Prompt

> Read the uploaded weather file and summarize it.

### Tool Sequence

```
weather
```

### Expected Output

- Dry bulb temperature
- Relative humidity
- Wind speed
- Global Horizontal Irradiance
- Direct Normal Irradiance

---

## 10. Extract Occupancy Information

### Example Prompt

> List all PEOPLE objects in the building.

### Tool Sequence

```
init
    ↓
occupancy
```

### Expected Output

- Occupancy schedules
- Occupancy calculation method
- Number of people
- Area per person

---

# Example Multi-Tool Workflow

## Prompt

> Load the building, summarize its occupancy configuration, run a baseline simulation, increase occupancy by 30%, rerun the simulation, compare annual electricity demand, and identify the hottest operating week.

## Expected Tool Sequence

```
init
    ↓
occupancy
    ↓
runperiod
    ↓
juzcsv (Baseline)
    ↓
altglo
    ↓
runperiod
    ↓
juzcsv (Modified)
    ↓
hourlycomplot
    ↓
weeksim
```

## Expected Output

- Occupancy summary
- Baseline simulation results
- Modified simulation results
- Energy comparison plots
- Weekly performance comparison

---

# Notes

- All simulation-based tools require a successfully initialized `idf_object`.
- `runperiod()` should be executed before every simulation when custom start/end dates are required.
- `altglo()` currently supports global modification of **PEOPLE** objects.
- `juzcsv()` generates the primary simulation output (`eplusout.csv`) used by downstream analytics and visualization tools.
- Visualization tools (`baseviz`, `weeksim`, `heatmapzone`, `hourlycomplot`) operate on EnergyPlus output data and should be executed only after a successful simulation.
