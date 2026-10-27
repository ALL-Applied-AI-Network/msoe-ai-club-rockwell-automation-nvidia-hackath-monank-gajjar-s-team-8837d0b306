---
name: memory
description: Project brain for the Rockwell Nvidia Physical AI Hackathon repo. Holds the project idea, problem statement, structure, data, decisions, progress and open items. Use it to recall project state before working, or to generate a handoff context block for another Claude session.
---

# Project Memory

This file is the single source of truth about the project. Read it fully before starting work. It is edited only through the `update-memory` skill (or by the user directly).

## Modes

1. **Recall** (default): read the sections below and use them as project context.
2. **Generate context**: when asked for a context for another Claude session, output a self-contained block (in a fenced markdown code block) containing: project overview, problem statement, structure, data notes, working rules, current status, key decisions, and next steps. Copy the facts from the sections below; do not invent or summarise away details. Tell the reader to treat it as background and to verify file names against the repo.

Facts marked `TBD` are unknown; never guess them. Ask the user or leave them as TBD.

## 1. Project Overview
- Name: rn-hackathon
- Event: Rockwell Automation × NVIDIA Physical AI Innovation Lab, Fall 2026 (four weeks). Run with the MSOE AI Club and UWM Connected Systems Institute (CSI).
- Goal of the event: pick one real process on CSI's testbed, make it measurably better, and show it working on real hardware.
- Repo: git, branch `master`, owner/git user MonankG. The team also gets a repository through the ALL AI Network dashboard.
- Role of assistant: acts as an assistant throughout the whole hackathon project.
- **Team:** "Monank Gajjar's team", 4 members (Monank is a UWM student; other members' schools TBD):
  - **Monank Gajjar** (lead developer): system integration, camera sync, color measurement pipeline.
  - **Malhar**: digital twin rendering and synthetic data (Isaac Sim / Replicator).
  - **Shubham**: segmentation model training (TAO) and TensorRT / DeepStream deployment.
  - **Ajay**: evaluation code, Cosmos Reason explanations, documentation.
- **Schedule:** kickoff Oct 8; team plan due Oct 14 (submitted); **final submission Sun Nov 1, 11:59 PM CT** (about a 5-minute video, a one-page summary, the repo link, references); **finals Thu Nov 5**, MSOE Diercks Hall, 5–8 PM.
- **What judges expect (written):** the solution running on real hardware (supervised test on the CSI line); a specific problem and why it matters; a business case for other manufacturers (cost today, savings, adoption effort); measured results against a baseline, with honest limits. Detailed criteria and weights are not yet published.
- **Briefing emphasis (verbal, pre-hackathon):** winning depends less on how good the solution is and more on whether it is **scalable**, works in **real time**, and has **low latency**.
- **Ground rule:** real line data is for looking, not training. Training data must come from the **digital twin** (reason: on a high-mix, low-volume line, new recipes have no real data before they run).
- **Organiser feedback on our plan (received ~Oct 15–20):** the problem is aligned with the hackathon goals. Make sure the scenario, recipe overview and **operator interactions match the current plant workflow** (key to competitiveness).
- **How Monank likes to learn:** step by step, in very simple language, with analogies, small chunks and quick check questions. He takes time to understand, so explain the intuition before details. A 6-step teaching plan was agreed (see section 8).

## 2. Project Idea
**Title (submitted):** *Right First Vial: Real-Time, GPU-Accelerated Prediction and Explanation of Color Carryover*

A real-time, scalable system for Dataset 2 (Fluid Mixing Station 3) that:
1. **Warns before filling:** scores each queued recipe transition for carryover risk, to prevent bad vials rather than only catch them.
2. **Detects** color and fill problems from the `vial_fill` camera with a **GPU segmentation model** (trained on digital-twin images via the TAO Toolkit, optimised with TensorRT, a few ms per frame) that isolates the liquid despite labels, glare and neighbouring vials. **DeepStream** processes the `vial_fill`, `tank_level` and `cap_seal` streams together, so one edge GPU can scale to multiple stations.
3. **Explains** the likely cause by linking the vial to the recipe sequence, dispense order and tank state. **Cosmos Reason** writes the explanation asynchronously, only for flagged vials, so it adds no latency.
- The expected color is predicted from a **mixing model plus the recipe**, so new recipes or tank colors need no retraining (the scalability claim).
- **Baseline:** a simple CPU color-measurement pipeline (fixed region plus color rules). The GPU system must beat it on missed defects at similar latency.
- **Evaluation:** leave-one-recipe-out; report missed defects and false alarms separately, explanation accuracy, and latency per vial.
- The carryover risk logic and expected-color calculation are simple logic (no GPU needed).
- **Runtime recipe source (changed after organiser answers):** our app **cannot read the recipe queue, PLC or any equipment data** at runtime. The plan is for the **operator to enter or select the upcoming recipe in our app**, mirroring how they already use the Batch Scheduler. ⚠️ Not yet confirmed as a team decision.
- **Expected color:** tank colors are learned from **real reference vials** (the clean last vial of each recipe), which organisers allow, then combined by mixing math. The `tank_level` camera checks tank state (e.g. low level, unexpected color).
- **Why not just hardware fixes or a color database (talking points for judges):**
  - Separate nozzles, flushing or discarding the first vial fix only carryover, cost hardware, downtime or waste, and miss other faults (wrong tank liquid, wrong amount, empty tank).
  - A hand-filled per-recipe color database breaks when recipes change often or tanks are refilled with different colors, and can't explain or warn.
  - We calculate the expected color instead ("calculator, not phone book").

## 3. Problem Statement
Dataset 2: Fluid Mixing Station (source: https://msoe-maic.com/library?article=Rockwell-NVIDIA-Fall-2026-Fluid-Mixing).
- Four tanks feed the vial filling station. A recipe sets the mL to draw from each tank, but the control system **does not know the actual color in each tank**. Fluids mix upstream of a **single shared nozzle**.
- **Carryover:** after a recipe change, residue from the previous recipe enters the **first vial** of the new batch and discolors it. The 2nd and 3rd vials come out clean, so the **last vial of each recipe** shows the true color. Example: Recipe 6 (30 mL clear T-100 + 10 mL orange T-200), then Recipe 7 (20 mL clear T-100 + 50 mL blue T-400): vial 7 is green, vials 8 and 9 are blue.
- The existing **Teledyne** vision system checks color and weight against a reference database (known color = pass). It doesn't check the intended recipe and doesn't warn before risky changes.
- Task: detect color or fill problems **and explain the likely cause** (recipe order, tank state, carryover). Flagging alone is not enough.
- **Metric priority:** positive class = defect. False negatives (missed defects) are the costliest error, so aim for high recall and report the false-alarm rate at the threshold that catches all known defects.

## 4. Data
- **Links:**
  - Sample data: https://drive.google.com/drive/folders/1qwV2BEz3ODX17kwNCr9M15DfkY8U_blQ?usp=sharing
  - Digital twin (datasets 1 and 2): https://drive.google.com/file/d/1nXRN_VZ_DKGS33jtELwn6z8WR8dbQfKe/view?usp=sharing
  - All lab data: https://drive.google.com/drive/folders/1GWuHV2WUNLztq_Fga6xBH7qFzhxDaac8?usp=sharing
- **Local location in repo:** planned `data/raw/` (keep out of git). Not downloaded yet as of 2026-10-27.
- **Organiser answers about data:**
  - All labels and details are on the website. **No PLC, batch-scheduler or Teledyne data exists.** Any extra data must be requested or collected by us.
  - All cameras were recorded by **one computer, so there is no clock offset** between `vial_fill`, `tank_level` and `cap_seal`.
  - There's **no organiser holdout set.** We collect our own, and judges evaluate on it.
  - **We define what a defect is** (color and fill deviations), aligned with the physical system.
  - Any calibration strategy, including real reference vials, is allowed.
- **Rule:** raw data must NEVER be modified. Write derived data elsewhere. Don't commit large datasets, checkpoints or keys.
- **Cameras:** 640×480; video at 30 fps; stills about every 2 s.
  - `vial_fill`: the vial-fill station;
  - `tank_level`: the four tanks;
  - `cap_seal`: the cap/seal station.
- **Videos:** five synchronised videos (IDs 1–5). Frames can be cross-referenced by timestamp.
- **Labels:** only 30 vials across 10 recipes (3 per recipe), on `vial_fill` only. Each has: recipe ID; per-tank volumes; total volume; dispense order; fill start and end times; per-frame "is filling" labels. Videos 1–4 and the `tank_level` and `cap_seal` views are unlabelled.
- **Tanks** (left to right in the `tank_level` view):
  - T-100: clear diluent; looks whitish.
  - T-200: orange; looks **pink** through the translucent wall.
  - T-300: **out of service, never used**, though it appears to hold greyish fluid.
  - T-400: blue; looks pale grey-blue.
- **Dispense order is fixed:** T-200 → T-400 → T-100, skipping unused tanks. Up to 3 recipes can be queued in the Batch Scheduler.
- **Line constraints:** ≤80 mL per vial; ≥10 mL of any selected component; ≤50 mL from T-200; never use T-300; no recipe changes without the plant manager.
- **Digital twin:** USD for Isaac Sim, covering the fill and cap/seal stations, with cameras matched to `vial_fill` and `cap_seal` and a render script. Tanks are **not** modelled and liquid color is approximate.
- **Organiser first step:** check that `vial_fill` frames match the closest `tank_level` and `cap_seal` frames. Record missing frames, clock offsets and drift before modelling.
- **Pitfalls (team analysis):**
  - no explicit defect label;
  - very little data;
  - a sim-to-real color gap;
  - fragile color measurement (transparent vial, white balance, small liquid region, labels, neighbouring vial);
  - non-linear subtractive color mixing;
  - unknown carryover strength;
  - washed-out tank colors;
  - garbled burned-in timestamps (use filenames, not OCR; there's no clock offset);
  - leakage between near-identical frames;
  - the explanation is scored separately.

## 5. Project Structure
```
rn-hackathon/
├── AGENTS.md                 # working rules for agents
├── README.md                 # one-line project description
├── .agents/skills/
│   ├── memory/SKILL.md           # this file (project brain)
│   ├── update-memory/SKILL.md    # procedure for updating this file
│   ├── generate-info/SKILL.md    # explain questions simply (analogies, keywords, visuals)
│   └── update-doc/SKILL.md       # add user-approved details to the Word doc
├── docs/
│   ├── Dataset2_Fluid_Mixing.docx  # team-facing Dataset 2 brief (maintained via update-doc)
│   └── project-overview.md         # earlier overview of all 3 datasets (no longer maintained)
├── explore/explore.py        # stage 1: exploration (empty)
├── preprocess/preprocess.py  # stage 2: preprocessing (empty)
├── features/features.py      # stage 3: feature engineering (empty)
└── model/model.py            # stage 4: modeling (empty)
```
Analysis proceeds in stages: explore -> preprocess -> feature engineering -> modeling. Code goes into the script of the relevant stage.

## 6. Working Rules (from AGENTS.md)
- Build on existing working code instead of replacing it unnecessarily.
- Avoid unnecessary functions, classes, abstractions.
- Production-style programming structures.
- Do not perform analyses, diagnostics, transformations or checks that were not requested.
- Never modify raw data.
- Skills for this project: create only `.agents/skills/<skill-name>/SKILL.md`; no `agents/openai.yaml` or other supporting files.

## 7. Tech Stack and Environment
- OS: Windows 11; shells: PowerShell / Git Bash. Language: Python (stage scripts).
- **Planned NVIDIA tooling:**
  - Isaac Sim + Replicator (synthetic data);
  - TAO Toolkit, via the NVIDIA TAO Skill Bank (segmentation training);
  - TensorRT (inference optimisation);
  - DeepStream (multi-camera real-time);
  - Cosmos Reason (explanations; free build.nvidia.com API first).
- **Resources GRANTED (as of 2026-10-27):**
  - **Brev: $500 credits.** Monank is captain with admin access. Invite teammates via Team → Generate link, as Member (or Maintainer if needed). Track spend in Billing. The Isaac launchable on **L40S costs $3.65/h**. Change GPU via launchable → Change → Change compute. RTX PRO 6000 / 4500 support Start/Stop. **Always stop instances when idle.**
  - **YAAD GPU Lab (UWM CSI):** a private team workspace on a **DGX Spark** (root, GPU, Docker), plus batch jobs on any of 8 Sparks. Accessed over VPN and SSH; credentials are in Monank's email. **Never put credentials in the repo or chat.** The Spark is ARM-based, so check that Isaac Sim, TAO, TensorRT and DeepStream run there; fall back to Brev.
  - **GB10 at CSI, already connected to the station cameras:** the live-test machine. Same platform family as the DGX Spark. Book through Shamar.
  - **build.nvidia.com:** usable now (Cosmos Reason API).
  - **AI agent plan:** the team received **ChatGPT Pro (Codex)**, not the requested Claude plan (as of 2026-10-27). Monank keeps using his own $20/month Claude plan in this session.
  - **Expected spend:** about $180 of $500 (rendering ~25 h × $3.65 ≈ $91; training ~10 h on RTX PRO 6000 ≈ $70; ~$20 buffer). **Time, not money, is the tightest resource.**
- **Compute originally requested (team plan):** Brev + UWM DGX Spark. About **$165 in Brev credits**:
  - Isaac Sim on L40S (AWS, Start/Stop), ~25 h, ~$75;
  - segmentation training on RTX PRO 6000, ~10 h, ~$60;
  - Cosmos Reason fallback, ~5 h, ~$15;
  - buffer.
- **Free compute:** own laptops (exploration, CPU baseline, latency benchmarks), DGX Spark (Isaac Sim setup, TensorRT/DeepStream edge latency tests; access details pending), build.nvidia.com API.
- **Brev practice:** stop instances between sessions; log GPU, runtime and credits for each run.
- **AI agent plans requested:** 1 × $100/month Claude (Monank); 2 × $20/month Claude (Malhar, Shubham); Ajay on free Gemini. Monank already pays for a $20/month Claude plan.
- **Mentor support requested:** NVIDIA, Rockwell Automation, Emulate3D / CSI.
- **Contacts (Slack):**
  - Shamar Webster, `#data-and-testbed-requests`: data, CSI visits, live tests.
  - Brett Storoe, Dr. Derek Riley, Tanner Cellio, `#dataset-2-fluid-mixing`: dataset questions.
  - Dan Schneider, `#compute-questions`: compute.

## 8. Progress Log
Newest last. Format: `YYYY-MM-DD - what was done`.
- 2026-10-09 - Repo initialised. Created stage folders with empty scripts (explore, preprocess, features, model). Created `memory` and `update-memory` skills.
- 2026-10-09 - Reviewed Dataset 1 (Final Vial Inspection). Created `generate-info` skill.
- 2026-10-12 - Reviewed Dataset 2 (Fluid Mixing) in depth: problem, carryover, tanks, data, pitfalls. Created `docs/project-overview.md`, `docs/Dataset2_Fluid_Mixing.docx` and the `update-doc` skill (repointed to the .docx).
- 2026-10-14 - Reviewed Dataset 3 (Grasping and Sorting) and compared all three. Committed to Dataset 2. Chose a GPU-based design. **Team plan submitted** (editable until 11:59 PM CT Oct 14). Word doc updated with the submitted plan.
- 2026-10-21 - Received organiser feedback and answers, the $500 Brev grant and YAAD DGX Spark access. Recorded them in the Word doc (§6, §9, §11, §11.1, §14, glossary).
- 2026-10-21 to 2026-10-27 - Teaching session with Monank, following the 6-step plan:
  1. Problem: ✅ done.
  2. Requirements: ✅ done.
  3. Dataset: ⏭️ skipped for now; teach it later or during exploration.
  4. Proposed solution: ✅ done.
  5. Resources vs solution: ✅ done.
  6. Project setup: next.

  Questions Monank raised: why not separate nozzles per tank; the current system only checks a color database; why not store each recipe's color in a database.
- 2026-10-27 - Timeline agreed (see section 11). Clarified the agent plan: the team got ChatGPT Pro, not Claude, so work continues in the current Claude session.

## 9. Key Decisions
- Stage-based folder layout mandated by AGENTS.md.
- Project state is stored inside this SKILL.md because skills may contain only SKILL.md.
- 2026-10-14: **Dataset 2 chosen.** Data is available now; the prevention-plus-explanation angle stands out; best fit for the scalable, real-time, low-latency emphasis. Dataset 3 data wasn't released yet and needs robotics skills; Dataset 1 is likely crowded.
- 2026-10-14: **GPU-based perception** (segmentation + TensorRT + DeepStream), with a CPU pipeline as the baseline. Reasoning: a CPU is fast for simple maths but not robust; neural segmentation is robust and needs a GPU to stay real-time and scale to many streams.
- 2026-10-14: **Brev request ~$165**, chosen over $70 (too tight; depends on DGX Spark timing and free API limits) and $175 (looked padded).
- 2026-10-14: AI agent plans: 1 × $100 + 2 × $20 Claude, with one member on free Gemini.
- 2026-10-14: Project title as in section 2.
- The Word doc `docs/Dataset2_Fluid_Mixing.docx` is the team-facing record. Edit it in place via `update-doc`; never regenerate it.
- 2026-10-27: **Two-version build plan.**
  - **v1 (CPU, today):** baseline liquid-region color and fill measurement, expected-color calculator, warn-before-filling rules, evaluation, Cosmos Reason explanations.
  - **v2 (GPU, Wednesday):** twin renders, segmentation model, TensorRT / DeepStream.
  - v1 alone is enough to go to CSI on Thursday.

## 10. Current Status
As of 2026-10-27: the plan has been reviewed by the organisers, and resources are granted ($500 Brev, DGX Spark workspace, build.nvidia.com, GB10 at CSI for the live test, ChatGPT Pro for agent work). **No data downloaded and no code written yet.** Final submission is **Sun Nov 1, 11:59 PM CT**, about 5 days away.

## 11. Open Questions / Next Steps
**All 7 organiser questions are answered** (see sections 1, 2, 4, 7 and the Word doc §14).

**Still open (team decisions):**
- How the recipe gets into our app at runtime (likely operator input), matching the current operator workflow.
- Our exact defect definition: color distance from expected beyond a threshold, plus under- and over-fill limits.
- Holdout collection plan: which recipe changes, how many vials, what to record.
- Compute split between the DGX Spark and Brev.
- Who on the team uses ChatGPT Pro (Codex), and for which tasks.

**Timeline (agreed 2026-10-27):**

| Day | Plan |
|---|---|
| Tue Oct 27 | Download data; build **v1** end to end on the provided data |
| Wed Oct 28 | Twin renders, segmentation training, TensorRT / DeepStream → **v2** |
| **Thu Oct 29 at CSI** | Record our holdout set (planned recipe changes); run v1/v2 on the GB10 |
| **Fri Oct 30 at CSI** | Iterate, final live test, measure latency on the GB10 |
| Sat–Sun Oct 31–Nov 1 | Final numbers, ~5-minute video, one-page summary, repo → **submit by Nov 1, 11:59 PM CT** |

**Immediate next steps:**
1. Continue in the current Claude session (no account switch needed).
2. Download the Dataset 2 sample data (Drive) into `data/raw/`, gitignored and never modified. The digital twin follows when the GPU work starts.
3. **Book CSI for Thu and Fri with Shamar Webster** (`#data-and-testbed-requests` or webste63@uwm.edu), including the test plan, duration, attendees and dates. Claude offered to draft this message.
4. Write v1 code into the stage scripts (`explore/`, `preprocess/`, `features/`, `model/`).
5. Set up VPN and SSH to the DGX Spark for the GPU steps.
6. Later: write the business case, record the video and write the one-page summary.
