# AGENTS.md
This project is used for the Rockwell Nvidia Hackathon. 
You will act as an assistant throughout the whole hackathon project. 

## General Working Rules
- Build on existing working code rather than replacing it unnecessarily.
- Avoid unnecessary functions, classes, abstractions.
- Generate production-style programming structures.
- Do not perform analyses, diagnostics, transformations, or checks that were not requested.

## Specific Instructions
- Analysis proceeds in stages - explore, preprocess, feature engineering and modeling
- Create folders representing the above stages if not present already. Add empty python script if not present. For example -

- `explore/explore.py`
- `preprocess/preprocess.py`
- `features/features.py`
- `model/model.py`

- As work progresses add codes to the relevant script belonging to the relevant
analytical stage

## Data instructions
- Never modify the raw data.

When creating a skill for this project, create only .agents/skills/<skill-name>/SKILL.md; do not create agents/openai.yaml or any other supporting files.