# WosHeteroStats Quick Reference Guide

## What is WosHeteroStats?
ETW post-processing tool to analyze **hybrid processor scheduling** on Windows.  
Converts ETL traces → CSV with scheduling statistics.

---

## Basic Command Syntax

```
WosHeteroStats.exe -i <input.etl> -o <output.csv> -big <mask> -small <mask> -smt <mask>
```

---

## Required Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Input ETL file path | `-i trace.etl` |
| `-o` | Output CSV file path | `-o results.csv` |
| `-big` | Bitmask for Big core LPs | `-big 0xC03C03` |
| `-small` | Bitmask for Small core LPs | `-small 0x3FC3FC` |
| `-smt` | Bitmask for HT-enabled LPs (0x0 if no HT) | `-smt 0x0` |

---

## Optional Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Time range (microseconds) | `-r 5000000 30000000` |
| `-ignoreParking` | Use if OS parking disabled | `-ignoreParking` |
| `-hgsplus` | Generate HGS+ stats (ADL+) | `-hgsplus` |
| `-PPM` | Generate PPM dump CSV | `-PPM` |
| `-cdie` | Compute die LP mask | `-cdie 0xFFFFFF` |
| `-socDie` | SoC die LP mask | `-socDie 0x0` |
| `/?` | Help info | `/?` |

---

## Default Example: ARL-S (8 Big + 16 Small, No HT)

### LP Topology Map
```
LP:  23 22 21 20 19 18 17 16 15 14 | 13 12 11 10 |  9  8  7  6  5  4  3  2 |  1  0
     ─────────────────────────────  ───────────   ─────────────────────────  ─────
              SMALL (8 LPs)          BIG (4 LPs)        SMALL (8 LPs)        BIG (4 LPs)
```

### Bitmasks
| Core Type | Hex Mask | Binary | LPs |
|-----------|----------|--------|-----|
| **Big** | `0xC03C03` | `1100 0000 0011 1100 0000 0011` | 0-1, 10-13, 22-23 |
| **Small** | `0x3FC3FC` | `0011 1111 1100 0011 1111 1100` | 2-9, 14-21 |
| **SMT** | `0x0` | `0` | None (no HT) |

### Command
```
WosHeteroStats.exe -i input.etl -o output.csv -big 0xC03C03 -small 0x3FC3FC -smt 0x0
```

---

## How to Calculate Your Own Bitmask

### Step 1: Identify LP positions for each core type
Check your system topology (use CPU-Z, HWiNFO, or Device Manager)

### Step 2: Set bits for each LP
Each LP = 1 bit position (LP0 = bit 0, LP1 = bit 1, etc.)

### Step 3: Convert binary to hex
Use Windows Calculator (Programmer mode) or online converter

### Example Calculation
```
LPs 0, 1, 10, 11, 12, 13, 22, 23 are Big cores

Binary (bit positions):
Bit:  23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
       1  1  0  0  0  0  0  0  0  0  1  1  1  1  0  0  0  0  0  0  0  0  1  1

Group into 4-bit nibbles:
1100 0000 0011 1100 0000 0011
 C    0    3    C    0    3

Result: 0xC03C03
```

---

## Common System Configurations

### Homogeneous 8-Core with HT
```
-big 0xFFFF -small 0x0 -smt 0xFFFF
```

### Hybrid 8 Big + 8 Small with HT on Big
```
-big 0xFFFF -small 0xFF0000 -smt 0xFFFF
```

### Hybrid 8 Big + 8 Small, No HT
```
-big 0xFF -small 0xFF00 -smt 0x0
```

### Lakefield (1 Big + 4 Small, No HT)
```
-big 0x10 -small 0xF -smt 0x0
```

### Meteor Lake (6 Big + 8 Small + 2 SoC Atoms with HT)
```
-big 0xffc03 -small 0x3003fc -smt 0xffc03 -cDie 0xFFFFFF -socDie 0x300000
```

---

## Key Output Statistics

| Statistic | What It Tells You |
|-----------|-------------------|
| **Concurrency** | How many LPs active simultaneously |
| **Big Core Idle + Small Active** | Scheduling inefficiency indicator |
| **Parking Stats** | Core parking behavior |
| **SMT Statistics** | HT sibling pairing efficiency |
| **Interrupt/DPC Hygiene** | Interrupt distribution across cores |

---

## Tips

1. **Verify masks** - Use Windows Calculator (Programmer mode) to double-check
2. **Check topology first** - LP assignments vary by platform
3. **Use `-r` flag** - Focus on specific time ranges for large traces
4. **Use `-ignoreParking`** - If your system has parking disabled

---

## Quick Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `Invalid command line arguments` | Typo in flags | Check for extra characters like `-big 8 -0xFF` |
| No output | Wrong mask values | Verify LP positions match your system |
| Missing stats | ETW provider not captured | Re-trace with correct providers enabled |

---

*Reference: Intel WosHeteroStats Tool Documentation*
