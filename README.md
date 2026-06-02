# CFD-Simulated Indoor Decaying Gas Plume Dataset

This dataset provides computational fluid dynamics (CFD) simulation snapshots of a decaying CO gas plume in a 10 × 10 m² indoor environment. It is used as the physical simulation backend for training and evaluating deep reinforcement learning agents on the indoor odor source localization (OSL) task.

---

## Directory Structure

```
dataset/
├── Barrier-free/          # Open-field scenario (no obstacles)
│   ├── 1.csv              # Earliest time step (highest concentration)
│   ├── 2.csv
│   ├── ...
│   └── 100.csv            # Latest time step (most diffused)
├── Obstacle/              # Cluttered scenario (3 rectangular obstacles)
│   ├── 1.csv
│   ├── ...
│   └── 100.csv
└── README.md
```

---

## Scenarios

| Scenario | Directory | Description |
|---|---|---|
| Barrier-free | `Barrier-free/` | Open 10 × 10 m² room, no physical obstructions |
| Obstacle | `Obstacle/` | Same room with three fixed rectangular obstacles |

**Obstacle positions (Obstacle scenario):**

| Obstacle | X range (m) | Y range (m) |
|---|---|---|
| 1 | 2.0 – 2.8 | 5.1 – 5.9 |
| 2 | 5.5 – 6.5 | 8.2 – 9.2 |
| 3 | 6.0 – 7.2 | 3.0 – 4.2 |

---

## File Format

Each CSV file follows the **ANSYS Fluent point-cloud export format**:

```
[Name]
Point Cloud 1

[Data]
X [ m ], Y [ m ], Z [ m ], co.Mass Fraction
1.99983597e-01, 1.76429749e-05, 4.99999970e-02, 9.09091011e-02
...
```

### Columns

| Column | Unit | Description |
|---|---|---|
| `X` | m | Horizontal position (0 – 10 m) |
| `Y` | m | Vertical position (0 – 10 m) |
| `Z` | m | Fixed at ≈ 0.05 m (sensor height, single horizontal slice) |
| `co.Mass Fraction` | dimensionless | Local CO mass fraction (gas concentration) |

---

## Physical Setup

| Parameter | Value |
|---|---|
| Room size | 10 × 10 m² |
| Spatial grid | 50 × 50 uniform grid, spacing ≈ 0.2 m |
| Sensing height (Z) | ≈ 0.05 m |
| Gas species | Carbon monoxide (CO) |
| Source position | (5.0, 5.0) m (room center) |
| Time snapshots | 100 per scenario |
| Temporal ordering | `1.csv` → earliest; `100.csv` → most decayed |
| Simulation tool | ANSYS Fluent (CFD) |

---

## Temporal Structure

The 100 files represent the time evolution of the plume after release. The plume is most concentrated in early snapshots and progressively disperses and decays toward snapshot 100. During a single RL training episode, the environment steps through snapshots sequentially (file index advances each time step), so the agent experiences a monotonically weakening concentration field within each episode.

---

## Usage

### Loading a single snapshot

```python
import pandas as pd

df = pd.read_csv(
    "dataset/Barrier-free/1.csv",
    skiprows=5,                        # skip Fluent header
    names=["x", "y", "z", "conc"]
)
print(df.head())
```

### Normalization (applied during RL training)

Raw CO mass-fraction values are globally min-max normalized to [0, 1] across **all** 100 snapshots at load time, so that the RL observation space is bounded and consistent across time steps.

```python
import numpy as np

all_values = np.concatenate([df["conc"].values for df in all_snapshots])
vmin, vmax = all_values.min(), all_values.max()
normalized = (df["conc"] - vmin) / (vmax - vmin)
```

---

## Citation

If you use this dataset, please cite the associated paper:

```
[To be filled in upon publication]
```

---

## License

The CFD simulation data was produced for academic research purposes.
Please contact the authors before redistribution or commercial use.
