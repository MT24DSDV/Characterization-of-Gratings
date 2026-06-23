# **Grating Characterization Toolkit**

This repository contains Python tools developed during my 2026 research internship
for the spectral characterization of optical gratings.

It includes:

- Reflection spectrum analysis
- Transmission spectrum analysis
- Transfer Matrix Method (TMM) simulation for grating modeling

The project is still evolving and may expand with additional analysis tools.

---

## **Table of Contents**

1. [Required Input Files](#1-required-input-files)
2. [Recommended Repository Structure](#2-recommended-repository-structure)
3. [File Naming Convention](#3-file-naming-convention)
4. [reflection\_analysis.py](#4-reflection_analysispy)
5. [transmission\_analysis.py](#5-transmission_analysispy)
6. [tmm.py](#6-tmmpy)
7. [Requirements](#7-requirements)
8. [Notes](#8-notes)

---

## **1. Required Input Files**

All analysis scripts expect `.csv` files exported directly from an Optical Spectrum
Analyzer (OSA).

The parser is designed for the standard OSA format containing a `[TRACE DATA]`
section with traces labeled **A–G**.

---

## **2. Recommended Repository Structure**


```
Grating-research/
├── reflection_analysis.py
├── transmission_analysis.py
├── tmm.py
├── data/
│ ├── reflection/
│ ├── transmission/
│ └── Example/
│ ├── comparison/
│ ├── cross/
│ └── full/
└── results/
├── reflection/
├── transmission/
└── tmm/
```


All scripts use **relative paths**, so the toolkit works on any machine without
modification.

---

## **3. File Naming Convention**

Measurement files follow this pattern:


```
SAMPLENAME_DD-MM-YY.csv
```

Examples:

```
BOX2_IN_OUT_10-06-26.csv
BOX1_0.050_10-06-26.csv
```

### Why this convention?

- The **sample name** appears first for quick identification.  
- The **date at the end** keeps files chronologically sorted.  
- Scripts automatically extract the date for labeling plots and outputs.  

If you use a different naming scheme, simply update the file name variables (`FILE`, `FILE_1`, etc.).  
The scripts do **not** depend on the date format.

### About the “box” terminology

In the original experiment, gratings were grouped into physical containers (“boxes”), and each OSA file contained multiple traces (A–G) corresponding to gratings in that box.

This is **only a naming convention**.  
You can replace it with:

- `Sample1`  
- `Setup_A`  
- `Test01`  
- anything that fits your workflow  

The scripts only rely on the file name you provide.

---
## **4. reflection_analysis.py**

Analyzes reflection spectra from OSA measurements.

### **Modes**

| Mode | Description |
|------|-------------|
| `single` | Analyze one file using a source and a reflected trace |
| `compare` | Compare two files side-by-side |
| `cross` | Overlay selected traces across multiple files |
| `box` | Compare all traces in one file against a reference trace |
| `full` | Plot the full spectrum of a file (all traces) |
| `single-box` | Compute reflection for each trace in a file against a shared source trace |

Set the mode at the top of the script:

```python
MODE = "single"   # or "compare", "cross", "box", "full", "single-box"
```

---

### **Mode Details**

#### `single`

Computes the reflection spectrum from one source trace and one reflected trace
in a single file.

**Configure:**

```python
FILE            = DATA_DIR / "SAMPLE1_0.050_10-06-26.csv"
TRACE_SOURCE    = "A"
TRACE_REFLECTED = "E"

COMPARE_TRACES  = True    # set False to skip
TRACE_COMPARE_1 = "D"
TRACE_COMPARE_2 = "E"
```

**Outputs:**

- Full spectrum plot (all traces)
- Source vs reflected power plot + reflection curve
- *(Optional)* Side-by-side trace comparison

---

#### `compare`

Compares reflection spectra from two separate files.

**Configure:**

```python
FILE_1            = DATA_DIR / "SAMPLE1_08-06-26.csv"
TRACE_SOURCE_1    = "A"
TRACE_REFLECTED_1 = "D"

FILE_2            = DATA_DIR / "SAMPLE2_08-06-26.csv"
TRACE_SOURCE_2    = "A"
TRACE_REFLECTED_2 = "E"
```

**Outputs:**

- Full spectrum plot for each file
- 3×2 side-by-side comparison figure (source, reflected, and overlay for each file)
- CSV for each file

**Typical use cases:**

- Same grating measured at different OSA resolutions
- Measurements taken at different times
- Forward vs backward coupling directions
- Gratings from different sample groups

---

#### `cross`

Overlays up to 4 individual traces drawn from any combination of files.
Useful for directly comparing specific gratings across measurements.

**Configure:**

```python
FILE_1     = "SAMPLE1_IN_OUT_10-06-26"
FILE_2     = "SAMPLE1_OUT_IN_10-06-26"
FIBER_TYPE = "GratingName"

CROSS_TRACES = [
    (DATA_DIR / f"{FILE_1}.csv", "B", "GratingName_0.05 - in->out"),
    (DATA_DIR / f"{FILE_2}.csv", "G", "GratingName_0.05 - out->in"),
    (DATA_DIR / f"{FILE_1}.csv", "C", "GratingName_0.5 - in->out"),
    (DATA_DIR / f"{FILE_2}.csv", "F", "GratingName_0.5 - out->in"),
]
```

Each entry in `CROSS_TRACES` is a tuple: `(file_path, trace_letter, label)`.

Labels containing `"in->out"` or `"out->in"` are automatically grouped into
separate overlay figures.

**Outputs:**

- `in->out` overlay plot
- `out->in` overlay plot
- Isolated subplots for each trace (split across two figures if more than 2 traces)

---

#### `box`

Computes the reflection of every listed trace against a single reference trace,
all from the same file. Produces an individual plot and CSV per trace.

**Configure:**

```python
BOX_FILE            = DATA_DIR / "SAMPLE1_OUT_IN_10-06-26.csv"
BOX_REFERENCE_TRACE = "A"

BOX_TRACES = [
    ("B", "Grating_1-0.050"),
    ("C", "Grating_1-0.500"),
    ("D", "Grating_2-0.050"),
    ("E", "Grating_2-0.500"),
    ("F", "Grating_3-0.050"),
    ("G", "Grating_3-0.500"),
]
```

Each entry is `(trace_letter, label)`.

**Outputs (per trace):**

- Reference vs trace power plot + reflection curve
- CSV with reflection and return loss values
- Console summary (min / max / mean)

---

#### `full`

Plots every trace in a file on a single figure.
Useful for a quick visual overview of a measurement.

**Configure:**

```python
FULL_FILE    = DATA_DIR / "SAMPLE1_OUT_IN_10-06-26.csv"
FULL_EXCLUDE = []    # e.g. ["F", "G"] to hide specific traces
```

**Output:**

- Single figure with all traces overlaid

---

#### `single-box`

Similar to `box`, but tailored for a single measurement session: computes the
reflection of each listed trace against a shared source trace,
generating one plot per grating.

**Configure:**

```python
SINGLE_BOX_FILE   = DATA_DIR / "SAMPLE1_OUT_IN_10-06-26.csv"
SINGLE_BOX_SOURCE = "A"

SINGLE_BOX_FIBERS = [
    ("B", "Grating_1_0.050"),
    ("C", "Grating_1_0.500"),
    ("D", "Grating_2_0.050"),
    ("E", "Grating_2_0.500"),
    ("F", "Grating_3_0.050"),
    ("G", "Grating_3_0.500"),
]
```

Each entry is `(trace_letter, grating_name)`.

**Outputs (per grating):**

- Source vs grating power (top panel)
- Reflection curve in dB (bottom panel)

---

### **Output Locations**

Results are saved under:

```
results/reflection/<mode>/<resolution_tag>/<file_tag>/
```

For example:

```
results/reflection/box/RES_TEST/SAMPLE1_OUT_IN_10-06-26/
├── Grating_1-0.050/
│   ├── Grating_1-0.050_reflection.csv
│   └── Grating_1-0.050_reflection.png
└── Grating_1-0.500/
    ├── Grating_1-0.500_reflection.csv
    └── Grating_1-0.500_reflection.png
```

---

### **Computed Values**

For all modes that compute reflection, the output CSV contains:

| Column | Description |
|--------|-------------|
| `wavelength_nm` | Wavelength axis (nm) |
| `source_dBm` | Source power (dBm) |
| `reflected_dBm` | Reflected power interpolated to source grid (dBm) |
| `reflection_dB` | Reflected − Source (dB) |
| `return_loss_dB` | −(Reflected − Source) (dB) |

---

## **5. transmission_analysis.py**

Analyzes transmission spectra and computes insertion loss.

### **Outputs**

- Full spectrum plot
- Source vs transmitted power comparison
- Transmission and insertion loss curves
- CSV file with computed values
- Console summary (min / max / mean transmission)

Example output location:

```
results/transmission/Example/
├── transmission_SAMPLE1_DATE.csv
└── transmission_SAMPLE1_DATE.png
```

---

## **6. tmm.py**

Simulates theoretical grating spectra using the Transfer Matrix Method.

### **Features**

- Reflection spectrum (linear and dB)
- Transmission spectrum (dB)
- Wavelength shift vs applied strain
- CSV export of simulated spectra
- CSV export of peak wavelength vs strain
- Sensitivity calculation

Results are saved in:

```
results/tmm/
```

---

## **7. Requirements**

- Python 3.10+
- numpy
- pandas
- matplotlib

Install dependencies:

```bash
pip install -r requirements.txt
```

---

## **8. Notes**

- All scripts use relative paths for portability across machines.
- Input files must be OSA-exported `.csv` files with a `[TRACE DATA]` section.
- The file parser caches loaded files to avoid re-reading the same file multiple
  times within a session.
- Naming conventions are flexible — adjust file name variables and labels to match
  your own workflow.
- The project is actively evolving; additional modes and analysis tools may be added.

