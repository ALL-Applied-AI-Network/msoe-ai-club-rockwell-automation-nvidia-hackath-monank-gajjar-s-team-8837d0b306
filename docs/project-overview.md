# Rockwell × NVIDIA Physical AI Hackathon: Project Overview

> The team's single reference for this project: the hackathon, the testbed, the datasets, our problem understanding, and the solution as it develops.
> Last updated: 2026-10-12

**Status:** Challenge not yet selected. Datasets 1 and 2 reviewed; Dataset 2 (Fluid Mixing) under detailed analysis. Dataset 3 not yet reviewed.

---

## 1. Hackathon at a Glance

| Item | Detail |
|---|---|
| Event | Rockwell Automation × NVIDIA Physical AI Innovation Lab (Fall 2026), four weeks |
| Organisers | Rockwell Automation, NVIDIA, MSOE AI Club, UWM Connected Systems Institute (CSI) |
| Goal | Pick one real process on CSI's testbed, make it measurably better, and show it working |
| Prizes | $5,000 cash pool, RTX 5080, RTX 5070, sponsor swag (split and categories TBD) |
| Overview page | https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026 |

### Schedule

| Date | Milestone |
|---|---|
| Thu, Oct 8 | Kickoff at Rockwell Automation HQ, Milwaukee |
| **Wed, Oct 14** | **Team plan due, 11:59 PM CT** |
| Sun, Nov 1 | Final submission due, 11:59 PM CT |
| Thu, Nov 5 | Finals at MSOE Diercks Hall, 5–8 PM |

### Team plan (due Oct 14)
A short resource request, not a graded paper. It asks for: team name and members; chosen dataset(s); short problem description; extra data needed and why; expected compute (Brev credits, UWM DGX Spark, MSOE Dell GB10 on Rosie, CSI workstation, build.nvidia.com); AI-agent credits or tools; mentor support wanted; questions for organisers.

### Final submission (due Nov 1)
- About a 5-minute video
- One-page project summary
- Link to the team repository
- References and credits for data, models, code and tools

### What judges expect
Judges are from Rockwell Automation, NVIDIA, MSOE and UWM. Detailed criteria and weights are still being finalised.
- **The solution running on real hardware**: the CSI production line or the FANUC arm (supervised test, booked through the data collection guide).
- **A specific problem**: a defect type, failure mode or slow manual step, and why it matters on this line.
- **The business case for other manufacturers**: cost today (scrap, rework, downtime, labour), what the solution saves, and what adoption takes (hardware, setup time, changeovers, operator trust).
- **Measured results against a baseline**, and honesty about where it still fails.

Sponsor emphasis: results that hold up on real equipment with clear limits matter more than a clean demo. Rockwell values AI that makes changeovers faster and more reliable and that operators can understand and trust.

### Ground rule for all datasets
Real line data is for **looking, not training**. Training data must be generated with the **digital twin**. On a high-mix, low-volume line, new products have no real images before commissioning, and collecting labelled real defects is costly and repeats at every changeover.

---

## 2. The Factory Testbed (CSI)

- A high-mix, low-volume (HMLV) demonstration plant built with Rockwell Automation hardware.
- It produces vials filled with mixed-colour fluid from custom recipes. Several robots move vials through filling, inspection and final placement.
- A separate FANUC robot cell moves and sorts blocks and other objects. It is **not** connected to the vial line and is the system behind Dataset 3.

---

## 3. The Three Challenges

Organisers recommend one strong solution around a single dataset.

| # | Challenge | One-line goal | Details |
|---|---|---|---|
| 1 | Final Vial Inspection | Find missing caps, crooked caps, damaged vials and unexpected empty positions after the robot fills a fixture | [Section 4](#4-dataset-1-final-vial-inspection) |
| 2 | Fluid Mixing Station 3 | Use synchronised cameras to detect fill and colour problems caused by recipes and fluid carryover, and explain the cause | [Section 5](#5-dataset-2-fluid-mixing-station) |
| 3 | Grasping and Sorting | Use a digital twin and multiple cameras to make the standalone FANUC arm grasp and sort objects more reliably | Not yet reviewed |

**Selected challenge:** TBD

---

## 4. Dataset 1: Final Vial Inspection

Source: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Vial-Inspection

### Problem
At the final station, vials are placed into two fixtures: **G** (good) and **S** (scrap). Use the overhead camera to check visible vial defects and fixture occupancy after each placement. **A missed defect counts more than a false alarm.**

### Why the current inspection is not enough
The upstream station inspects vials from the side and routes them to G or S, but it misses some cap defects. A cap can also move after inspection, and the robot can miss a pick, damage a vial or misplace it without noticing.

### Fixtures
- A fixture is the plate that holds vials in fixed, numbered holes. Each has **30 positions** (6 columns × 5 rows).
- Numbering: **01 is top-right**. Positions run down the rightmost column (01–05), then the next column left (06–10), ending at the leftmost column (26–30). This is also the robot's fill order.
- The plates also have smaller holes between positions. Only the 30 numbered apertures count.
- An internal gap can mean a missed placement. Trailing empty positions may just be waiting for vials, so use a placement count or an order-complete signal where available.

### Labels (per target position, not per image)

| Label | Meaning |
|---|---|
| `pass` | Visible cap and body look acceptable (says nothing about hidden properties) |
| `missing_cap` | Vial present with an open mouth; don't confuse the clear body with an empty hole |
| `crooked_cap` | Cap visibly tilted or not seated; a leaning vial alone doesn't count |
| `damaged_vial` | Target body visibly damaged; overlapping clear bodies may need full-scene context |
| `no_vial` | Aperture empty; whether it's an error depends on fill progress |

### Data provided
- Full overhead images of both fixtures, named like `tray_000001.jpg`.
- Labelled and unlabelled crops per position, plus fixture coordinates and position IDs.
- A browser labelling tool (review JSON export). Its snapshot shows 840 positions = **14 images × 60 positions**. It also has a "needs review" status and a "closest reference" appearance distance.
- A digital twin (shared with Dataset 2).

### Observations from the images
- The scrap fixture is messy: vials lean, overlap and have transparent bodies.
- Crops include neighbouring vials; the prediction must follow the target aperture.
- The camera is wide-angle with visible distortion near the edges, and lighting varies (e.g. a warm-light calibration frame).

### Suggested approaches (from organisers)
- Generate synthetic data with Isaac Sim + Replicator.
- Visual ChangeNet (TAO Toolkit) compares against a golden image and outputs PASS / NO_PASS; trained via the NVIDIA TAO Skill Bank.
- Cosmos Reason for in-context inspection.
- Fill-pattern logic.
- DEFT AOI workflow for iterating on errors.

### Evaluation requirements
- Held-out test set; per-class results.
- False alarms and missed defects reported separately, with false negatives per defect class.
- False-alarm rate at the threshold that catches every known defect.
- Test on complete fixture scenes, not only crops.

### Links
- Digital twin (datasets 1 and 2): https://drive.google.com/file/d/1nXRN_VZ_DKGS33jtELwn6z8WR8dbQfKe/view?usp=sharing
- Sample data: https://drive.google.com/drive/folders/1Vl5qMLvUZDeWYNaFvnN8JbIqesjOKkWN?usp=sharing

---

## 5. Dataset 2: Fluid Mixing Station

Source: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Fluid-Mixing

### 5.1 Problem
Four tanks feed the vial filling station. A recipe sets how many mL to draw from each tank, but the system **does not know the actual colour in each tank**. Fluids mix upstream of a **single shared nozzle**, so fluid left from the previous fill can carry into the next recipe.

**Goal:** build a system that detects colour or fill problems and **explains the likely cause**. It should link what the cameras saw to the recipe order, tank state or colour carryover. Flagging that a vial looks different is not enough.

### 5.2 Carryover
- When the recipe changes, residue from the previous recipe's mixture in the shared path enters the **first vial** of the new batch, so that vial is discoloured.
- The **2nd and 3rd vials** come out clean. The **last vial of each recipe** reliably shows the recipe's true colour.
- **Example:** Recipe 6 (30 mL clear T-100 + 10 mL orange T-200) runs, then Recipe 7 (20 mL clear T-100 + 50 mL blue T-400). Vial 7, the first of Recipe 7, comes out **green**; vials 8 and 9 come out **blue**.

### 5.3 Equipment and recipes

| Tank | Position (in the `tank_level` view) | Fluid | Appearance on camera |
|---|---|---|---|
| T-100 | leftmost | clear diluent | whitish |
| T-200 | 2nd | orange | pink band (washed out by the translucent tank wall) |
| T-300 | 3rd | **out of service** | greyish; appears to hold fluid but is unused |
| T-400 | rightmost | blue | pale grey-blue band |

- Tanks are identified by number, not colour, because colours can change.
- **Dispense order is fixed:** T-200 → T-400 → T-100, skipping unused tanks. This order decides which fluid passes through the shared path last.
- Up to 3 recipes can be queued in the Batch Scheduler. The labelled data covers 10 recipes.

**Operating constraints (when collecting data on the line):**
- At most 80 mL per vial.
- Components from 0–80 mL, but at least 10 mL of any selected component.
- At most 50 mL from T-200 (more can fault the system).
- Never use T-300.
- No recipe changes without the plant manager's guidance.

### 5.4 Why the current inspection is not enough
A Teledyne vision system checks vial colour and weight before the final station. It compares the colour against a reference database (known colour = pass, unknown = fail). It does not check whether the vial matches the **intended recipe**, and it does not warn that an upcoming recipe change is risky.

### 5.5 Data provided
- Three fixed cameras at 640×480: video at 30 fps, stills about every 2 s.
  - `vial_fill`: the vial-fill station
  - `tank_level`: the four tanks
  - `cap_seal`: the cap/seal station
- Five synchronised videos (IDs 1–5). Frames can be cross-referenced by timestamp.
- **Labelled data:** 30 vials across 10 recipes (3 per recipe), on `vial_fill` only. Each vial has: recipe ID; per-tank volumes; total volume; dispense order; fill start and end times; per-frame "is filling" labels.
- Everything else is unlabelled, including videos 1–4 and the `tank_level` and `cap_seal` views.
- A digital twin (USD for Isaac Sim) of the fill and cap/seal stations, with cameras matched to `vial_fill` and `cap_seal` and a render script. **Tanks are not modelled and liquid colour is approximate.**
- Barcodes and labels may be present, in inconsistent orientations. Production does not rely on them.

### 5.6 Links
- Digital twin (datasets 1 and 2): https://drive.google.com/file/d/1nXRN_VZ_DKGS33jtELwn6z8WR8dbQfKe/view?usp=sharing
- Sample data: https://drive.google.com/drive/folders/1qwV2BEz3ODX17kwNCr9M15DfkY8U_blQ?usp=sharing

### 5.7 Organiser guidance
- **First step:** check that a `vial_fill` frame can be matched to the closest `tank_level` and `cap_seal` frames. Record missing frames, clock offsets and timestamp drift before modelling.
- **Starter ideas:**
  - Simple colour and fill-level measurement first.
  - Link the three cameras.
  - Use the recipe sequence.
  - Cosmos Reason with example fills.
  - Synthetic data from the twin.
  - Label videos 1–4 to grow the test set.
- **Testing:**
  - Split by production run or time window.
  - **Leave-one-recipe-out** evaluation.
  - Report false alarms and missed defects separately.
  - Evaluate the claimed cause **separately** from detection.

### 5.8 Problem understanding (team analysis)
The system must:
1. Detect colour or fill problems in a vial.
2. Explain the likely cause: carryover from the previous recipe, the wrong recipe, or tank state (e.g. low level or an unexpected colour).
3. Ideally, warn before filling that an upcoming recipe transition is risky.

### 5.9 Potential pitfalls (team analysis)
1. **No explicit defect label.** Labels give the recipe and volumes, not good/bad. "Discoloured" has to be derived (first vial of a batch = likely contaminated). There are probably no labelled fill-level defects.
2. **Very little data.** Only 2 clean references per recipe and about 10 carryover examples in total, so any metric will be noisy.
3. **Sim-to-real colour gap.** We must train on the twin, but its liquid colour is approximate and the tanks aren't modelled. Colour, the main signal, is where the twin is weakest.
4. **Fragile colour measurement:**
   - The transparent vial shows the background through the liquid.
   - Warm white balance and lighting changes shift colours.
   - The visible liquid region is small.
   - Fill height changes the apparent colour.
   - Barcode labels can cover the liquid.
   - A neighbouring vial is always in frame.
5. **Non-linear colour mixing.** Dyes mix subtractively (blue + orange residue gives green), so expected colour isn't an RGB average.
6. **Unknown carryover strength.** It may depend on the last-dispensed fluid, volumes and timing. With about 10 transitions, explanations will likely be qualitative.
7. **Tank colour inference.** Tank colours seen through the translucent walls are washed out (orange looks pink). T-300 holds fluid but is unused.
8. **Timestamp sync.** Burned-in timestamps are garbled (double overlay), so use filenames rather than OCR. Stills every 2 s can miss events.
9. **Leakage.** Neighbouring frames are near-identical. Use run, time-window or leave-one-recipe-out splits; 10 folds means high variance.
10. **Explanation is scored separately.** We need a way to verify the cause the system claims (e.g. correct prior recipe or tank).
11. **Line safety constraints** apply to any data collection (see 5.3).

### 5.10 Candidate solution sketch (not decided)
1. **Baseline:** segment the vial's liquid region; measure colour in Lab or HSV plus fill height; compare with the recipe's reference colour (the last vial of each recipe).
2. **Context layer:** use the recipe sequence (previous recipe and dispense order) plus tank state from `tank_level` to produce rule-based cause explanations.
3. **Reasoning layer:** Cosmos Reason with example frames plus recipe context for human-readable explanations.
4. **Pre-fill warning:** a carryover risk score for each upcoming recipe transition.
5. **Twin data:** render with varied colour, fill level and lighting for robustness; calibrate twin colours to the real reference vials.

---

## 6. Dataset 3: Grasping and Sorting

Source: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Grasping-Sorting

Not yet reviewed. Summary from the overview: use a digital twin and multiple cameras to make the standalone FANUC arm grasp and sort objects more reliably.

---

## 7. Our Solution

**Selected challenge:** TBD

| Item | Detail |
|---|---|
| Problem statement | TBD |
| Approach and architecture | TBD |
| Synthetic data generation | TBD |
| Model(s) | TBD |
| Baseline | TBD |
| Results vs baseline | TBD |
| Known failure cases and limits | TBD |
| Real-hardware test | TBD |

---

## 8. Business Case

| Item | Detail |
|---|---|
| Cost of the problem today (scrap, rework, downtime, labour) | TBD |
| What the solution saves or improves | TBD |
| Adoption requirements (hardware, setup time, changeovers, operator trust) | TBD |

Context from the organisers: in the 2020 Annual Business Survey, only about 3% of manufacturers used AI for automated inspection. A NIST survey estimates defective products cost US discrete manufacturers $32–58B per year.

---

## 9. Compute and Resources
- Compute guide: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Compute
  - Options: Brev (credits spent while an instance is running; RTX PRO 6000 preferred, L40S cheaper), UWM DGX Spark, MSOE Dell GB10 (Rosie), CSI workstation, build.nvidia.com.
- Tools and resources (includes an Isaac Sim on Brev tutorial): https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Resources
- Collect and test your own data: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Data-Collection
- Coding agents: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Coding-Agents
- All lab data: https://drive.google.com/drive/folders/1GWuHV2WUNLztq_Fga6xBH7qFzhxDaac8?usp=sharing
- Key tools: Isaac Sim + Replicator, TAO Toolkit / Visual ChangeNet, DEFT AOI, Cosmos Reason.

### Getting help (hackathon Slack)
- More data, CSI visits or physical testing: Shamar Webster, `#data-and-testbed-requests`.
- Line or dataset questions: Brett Storoe, Dr. Derek Riley or Tanner Cellio, in the relevant dataset channel.
- Brev or compute: Dan Schneider, `#compute-questions`.
- Website, signup, team or Slack access: Brett Storoe, `#hackathon-general`.

---

## 10. Decision Log

| Date | Decision | Reason |
|---|---|---|
| | | |

---

## 11. Open Questions
- Which challenge do we choose? (The team plan is due Oct 14.)
- Dataset 2: what exactly counts as a defect (any carryover vial, or only colour beyond a threshold)? Are fill-level faults in scope?
- Dataset 2: is a held-out real test set coming, or do we hand-label videos 1–4?
- How strictly is "train only on the twin" applied to colour baselines that use real reference vials?

---

## 12. Glossary

| Term | Meaning |
|---|---|
| HMLV | High-mix, low-volume: many product variants in small batches, with frequent changeovers |
| Digital twin | A simulated copy of the physical station used to generate labelled training data |
| Fixture | A plate that holds vials in fixed, numbered positions |
| Vial | A small clear tube with a cap, filled with coloured fluid |
| Carryover | Residue from the previous recipe in the shared mixing path that discolours the next vial |
| Diluent | The clear fluid (T-100) used to dilute colours |
| Golden image | A known-good reference image used for comparison |
| False negative (FN) | A missed defect; the worst outcome in judging |
| False positive (FP) | A false alarm; a good item flagged as defective |
| Recall | Defects caught ÷ all real defects |
| Leave-one-recipe-out | Hold out all vials of one recipe for testing, train or reference on the rest, and repeat for each recipe |
| Cosmos Reason | NVIDIA vision-language model that can reason over images or video with a text prompt |
| Visual ChangeNet | TAO Toolkit model that compares an image with a golden image and outputs PASS / NO_PASS |
| Isaac Sim / Replicator | NVIDIA simulation platform and its synthetic data generation tool |

---

## 13. Change Log

| Date | Change |
|---|---|
| 2026-10-12 | Document created: hackathon overview, testbed, Datasets 1 and 2 details, Dataset 2 problem analysis and pitfalls |
