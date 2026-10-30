# Design, Simulation and Manufacturing of an Injection-Molded Part
### Game Boy Advance SP top shell: a Design-for-X case study

Work project for the **Digital Manufacturing** course (Management Engineering).

The goal of the course is to show how **DFX (Design for X)** methods and digital tools can cut product cost and shorten time-to-market. In this project we followed one plastic part through the whole digital chain: from reverse engineering and structural validation, through injection molding simulation and cost estimation, to mold design and CNC milling simulation.

> **Component:** the top shell of a Game Boy Advance SP, the area around the D-pad and the A/B buttons.
> **Final objective:** find the design and process configuration with the **lowest overall cost** (material + processing + mold), while keeping the part structurally sound and manufacturable.

![hero picture of the shell](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/hero.png?raw=true)

---

## Table of contents

1. [Project overview](#1-project-overview)
2. [Workflow](#2-workflow)
3. [Engineering design lab](#3-engineering-design-lab)
4. [Industrialization lab](#4-industrialization-lab)
5. [Cost analysis](#5-cost-analysis)
6. [Manufacturing lab: mold milling](#6-manufacturing-lab-mold-milling)
7. [Key results](#7-key-results)
8. [Repository structure](#8-repository-structure)
9. [Tools and how to reproduce](#9-tools-and-how-to-reproduce)
10. [Team](#10-team)

---

## 1. Project overview

| | |
|---|---|
| **Course** | Digital Manufacturing, Management Engineering |
| **Part** | Game Boy Advance SP top shell (button area) |
| **Process** | Injection molding, two-plate mold |
| **CAD / CAE / CAM** | Autodesk Fusion (Design, Simulation, Injection Molding Simulation, Manufacture) |
| **Cost model** | Excel injection molding cost estimator (`.xlsm`) |

The part was chosen because it fits the course constraints: thin-walled (1 to 3 mm), small (max size 10 to 20 cm, not too tall, so the mold can be milled), and it can be reinforced with ribs.

### Design-for-Manufacturing choices built into the part

- **Draft angles** on every surface formed by the fixed mold plate, so the part can be ejected without overhangs.
- **Uniform wall thickness** across the component, to avoid defects such as warpage and short shots.
- **Fillets** instead of sharp edges, so the mold can actually be milled.
- **Fully parametric CAD**, driven by the main wall thickness, so any thickness change propagates to the whole model (button holes, bosses, rib thickness, and so on).

---

## 2. Workflow

```
Part selection
   │
   ▼
Reverse engineering ─► Parametric CAD (thick-walled, no ribs)
   │
   ▼
Load case + structural requirement (max displacement 0.1 mm)
   │
   ▼
Mesh convergence study ─► Material comparison (ABS / PC / PP)
   │                         thickness tuned to equalize deflection
   ▼
Cost comparison ─► choose the cheapest material
   │
   ▼
Thickness reduction + reinforcing ribs (stay under 0.1 mm)
   │
   ▼
Injection molding simulation ─► tune process parameters / gate position
   │                            (if not manufacturable: adjust, or thicken slightly)
   ▼
Cost estimate (volume, number of cavities)
   │
   ▼
Mold design (two plates) ─► CAM: milling strategy and simulation
```

---

## 3. Engineering design lab

### 3.1 Reverse engineering

The CAD model was rebuilt from the real GBA SP case, keeping the real dimensions and ratios and the original features: button holes, hinge, speaker holes and screw bosses.

![reference model to CAD](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/reference_to_cad.png?raw=true)

### 3.2 Parametric design

All dimensions are linked to the **main shell thickness**, so the model stays consistent when the thickness changes. For example, the D-pad area is defined by a formula that ties its geometry to the shell thickness and to the dimension ratios of the reference part. Rib thickness is also tied to the wall thickness.

### 3.3 Load case and constraints

| Item | Value |
|---|---|
| Nominal force | up to 5 N |
| Extreme force | 10 N |
| Maximum allowed displacement | **0.1 mm** (chosen so the product feels sturdy) |
| Applied load | 10 N pressed on both button areas (D-pad and A/B) |
| Constraints | fixed on the screw bosses |

The load is deliberately demanding (extreme case), so that ribs and stiffer materials are actually justified.

### 3.4 Mesh sensitivity analysis

Displacement and maximum von Mises stress were evaluated for mesh sizes from 10% down to 2%.

- Displacement stays roughly between 0.099 and 0.102 mm across all mesh sizes.
- Maximum stress is unstable for coarse meshes and plateaus between **5% and 3%**.
- **4%** was selected: converged results at a reasonable simulation time.

![mesh convergence graphs](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/mesh_convergence_charts.png?raw=true)

### 3.5 Material selection

Three candidates were compared. For each one, the thickness was tuned so that all materials reach the same maximum deflection, using ABS as the reference. A stiffer material allows a thinner wall.

| Material | Notes | E (GPa) | Thickness | Volume | Cost (1 cavity, 1M parts) |
|---|---|---|---|---|---|
| **ABS** | reference material, ductile | 2.24 | 2 mm | 15.35 cm³ | **0.345 $/part** |
| PC | stiff, brittle, more expensive | 2.275 | 2 mm | 15.35 cm³ | 0.378 $/part |
| PP | less performant, cheaper | 1.34 | 2.6 mm | 19.45 cm³ | 0.538 $/part |

**ABS** gave the lowest overall cost. PP is cheaper per kg but needs a much thicker wall, so it ends up the most expensive part. PC is barely stiffer than ABS, so its higher price is not repaid by a thinner wall.

### 3.6 Stress / displacement analysis

With ABS at 2 mm, the displacement peaks at **0.102 mm**, localized around the D-pad cutout. That is slightly over the 0.1 mm limit, so the design needed improvement.

![displacement picture](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/displacement_plot.png?raw=true)

### 3.7 Design optimization

Two strategies were tested to bring the displacement back under the limit while reducing wall thickness:

1. **Thickness variation:** 2 mm, 1.5 mm and 1.25 mm. **1.5 mm** was used for the final model.
2. **Additional ribs:** four reinforcing ribs connecting the D-pad ring to the screw bosses, with rib thickness parametrically linked to the wall thickness.

Rib layouts were refined by trial and error: some early layouts actually performed *worse* (peak displacement around 0.147 to 0.149 mm), and the final layout lowered the localized displacement.

![displacement final design](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/displacement_at_final_design.png?raw=true)

---

## 4. Industrialization lab

### 4.1 Injection molding simulation

Simulated with Autodesk Fusion's injection molding tool, using the ABS process windows as reference (melt 230 to 260 °C, mold surface 40 to 80 °C).

**Baseline (automatic settings):** melt 245 °C, mold 60 °C, injection time 0.74 s.
The part is **easy to fill**, but shows warpage at the corners and a non-flat surface (about 8% of the area out of the flatness tolerance, up to 0.25 mm), which would prevent proper assembly. Warpage up to ~0.3 mm is not ideal but acceptable; flatness is the main issue.

### 4.2 Iterations and results

| # | Change | Melt / Mold | Inj. time | Flat area | Max flatness dev. |
|---|---|---|---|---|---|
| 0 | Baseline, no side ribs | 245 / 60 °C | 0.63 s (auto) | 93.96% | 0.236 mm |
| 1 | + side ribs | 245 / 60 °C | 0.63 s (auto) | 95.4% | 0.219 mm |
| 2 | Temperature sweep (3×3 grid) → best case | 260 / 80 °C | 0.73 s (auto) | 95.6% | 0.210 mm |
| 3 | Gate moved slightly to the right, towards the last-filled region | 260 / 80 °C | 0.73 s (auto) | 96.2% | 0.205 mm |
| 4 | Two injection points | 260 / 80 °C | 0.41 s (auto) | 84% | 0.28 mm (worse) |
| 5 | **Final:** new gate position, injection time specified manually | **260 / 80 °C** | **0.63 s** | **96.5%** | **0.203 mm** |

The temperature sweep covered mold temperature {40, 60, 80} °C × melt temperature {230, 245, 260} °C.

Takeaways:
- Higher melt and mold temperatures reduced warpage.
- Side ribs improved flatness.
- A single, well-placed gate beat a two-gate layout.
- The first gate position (middle of the part) filled the hinge area last, so moving it closer to that region helped symmetry.

![flatness plane](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/flatness_plane.png?raw=true)

### 4.3 Mold design

Final mold: **two-plate mold**, with the injection point along the upper edge of the part to promote uniform cavity filling. Plate 1 forms the button side; plate 2 forms the hinge/rear side.

![two plates design](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/two_plates.png?raw=true)

---

## 5. Cost analysis

Total part cost is the sum of three contributions, computed in the Excel estimator (`cost_model/`):

| Contribution | Driven by |
|---|---|
| **Material cost** | part volume, so wall thickness and ribs |
| **Processing cost** | cycle time, so mainly wall thickness |
| **Mold cost** | mold machining, size and number of cavities |

A yield of 0.95 is applied.

### Single-cavity baseline (ABS)

| Item | Value |
|---|---|
| Mold cost (total) | 3,133 $ |
| Mold cost per part | 0.006 $ |
| Processing cost per part | 0.284 $ |
| Material cost per part | 0.048 $ |
| **Total part cost** | **0.345 $/part** |

### Production volume vs. cost

Unit cost falls quickly as volume grows, as the mold cost is spread over more parts, and flattens out at about **0.3437 $/part** from roughly **1.3 million parts**.

| Production volume | Part cost ($) |
|---|---|
| 10,000 | 0.622 |
| 50,625 | 0.419 |
| 75,938 | 0.402 |
| 113,906 | 0.371 |
| 576,650 | 0.355 |
| 864,976 | 0.355 |
| **1,297,463** | **0.344** |
| 1,946,195 | 0.344 |
| 2,919,293 | 0.344 |

### Number of cavities vs. cost

More cavities cut processing cost per part, but a bigger mold costs more. The optimum is a balance.

| Cavities | Total cost per part ($) |
|---|---|
| 1 | 0.6839 |
| 2 | 0.4325 |
| 4 | 0.3121 |
| 6 | 0.2765 |
| 8 | 0.2619 |
| 10 | 0.2557 |
| **12** | **0.2536** |
| 16 | 0.2574 |

**Optimum: 12 cavities.**

| Final configuration | |
|---|---|
| Mold cost | 12,418 $ |
| Part (processing) cost | 0.128 $/part |
| Material cost | 0.048 $/part |

---

## 6. Manufacturing lab: mold milling

One of the mold blocks was machined in simulation using Autodesk Fusion Manufacture, with tool feeds, speeds and depths of cut set per operation. The hardest areas of the mold are the hinge region, the chamfers and fillets, and the small features (speaker holes, small bosses).

| Step | Operation | Tool | Time |
|---|---|---|---|
| 1 | Face milling | Ø50 mm face mill | 1:31 min |
| 2 | 2D contour | Ø18 mm flat endmill | 0:53 min |
| 3 | 2D adaptive | Ø2.5 mm flat endmill | 4:29 min |
| 4 | Pocketing / clearing | Ø2.5 mm flat endmill | 6:07 min |
| 5 | 2D contour | Ø2.5 mm flat endmill | 0:29 min |
| 6 | 2D adaptive | Ø2.5 mm flat endmill | 7:15 min |
| 7 | 3D parallel | Ø2 mm bullnose endmill | 33:31 min |
| 8 | 3D parallel | Ø2 mm bullnose endmill | 25:04 min |
| 9 | 3D parallel | Ø2 mm bullnose endmill | 9:42 min |
| 10 | 3D scallop | Ø1 mm bullnose endmill | 3:02 min |
| 11 | 3D ramp | Ø1 mm bullnose endmill | 0:08 min |
| 12 | 3D flow | Ø1 mm bullnose endmill | 2:53 min |
| 13 | 2D contour | Ø1 mm flat endmill (optional) | 0:01 min |
| 14 | Face milling (opposite side) | Ø50 mm face mill | 3:23 min |

**Total machining time: 1 h 38 min 28 s** (as reported by the CAM simulation).

![mold milling procedure](https://github.com/Tzomy/Digital_manufacturing_project/blob/docs/images/mold_milling.png?raw=true)

---

## 7. Key results

- **Material:** ABS, the cheapest overall solution once thickness is tuned to equal stiffness.
- **Geometry:** 1.5 mm main wall with 4 reinforcing ribs, instead of the initial 2 to 3 mm solid wall.
- **Mesh:** 4% mesh size gives converged results at acceptable computational cost.
- **Process:** melt 260 °C, mold 80 °C, injection time 0.63 s, single gate on the upper edge (shifted slightly right). Flat area **96.5%**.
- **Production:** two-plate mold with **12 cavities**, optimal for high volumes (≈1.3 M parts and above).
- **Cost:** from 0.345 $/part (single cavity baseline) down to **≈0.25 $/part total** in the final configuration.
- **Machining:** the mold block can be milled in about 1 h 38 min.

---

## 8. Repository structure

```
.
├── README.md
├── simulation/
│   ├── Material_study.f3d               # Different materials thickness optimization
│   ├── Final_injection_molding.f3d      # Fill, warpage, temperature sweep, gate position best results
│   └── Mold_machining_done.f3d          # Fusion Manufacture setups, toolpaths, post-processed code
├── cost_model/
│   └── Injection_Molding_cost_estimator.xlsm
├── docs/
│   ├── Digital_manufacturing_presentation.pdf
│   └── images/
└── LICENSE
```

---

## 9. Tools and how to reproduce

**Software**
- Autodesk Fusion (Design, Simulation, Injection Molding Simulation, Manufacture)
- Microsoft Excel (macro-enabled `.xlsm`, so enable macros)

**Steps**
1. Open the part in `cad/part/`. Change the `thickness_shell` user parameter to regenerate the model.
2. Run the structural study in `simulation/structural/` (load case: 10 N on the buttons, constraints on the bosses, mesh 4%).
3. Open `simulation/injection_molding/` and run the study with the parameters of the final configuration (ABS, 260 °C melt, 80 °C mold, 0.63 s).
4. Enter the resulting volume and cycle time in `cost_model/Injection_Molding_cost_estimator.xlsm` to reproduce the cost curves.
5. Open the mold in `cad/mold/` and the setups in `manufacturing/cam/` to replay the milling simulation.

---

## 10. Team

- Tommaso Pintonato
- Muhammad Reza
- Alberto Borgo
- Luca Tarantini

Course: Digital Manufacturing, Management Engineering.
