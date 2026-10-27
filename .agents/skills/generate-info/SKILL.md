---
name: generate-info
description: Answer a question in simple terms, using a real-life analogy when one fits, explaining any technical keywords, and adding visuals (diagrams, images, animations) that make the answer easier to understand. Use when the user asks to explain or understand a concept, term, result or piece of the project.
---

# Generate Info

Explain the question so that a non-expert understands it on the first read.

## Answer Structure

1. **Short answer**: 1-3 plain sentences that answer the question directly.
2. **Analogy**: one real-life analogy, only if a relevant one exists. Map each part of the analogy to the real concept (e.g. "the chef = the model, the recipe = the weights"). Say where the analogy stops being accurate. Skip this section if no honest analogy fits.
3. **How it works**: a simple step-by-step explanation. Short sentences, no jargon left unexplained.
4. **Keywords**: every technical term used in the answer, each with a one-line plain-language definition. Use a short list or table.
5. **Visual**: at least one visual when it helps understanding (see below). Skip only for questions where a visual adds nothing.
6. **In the project** (optional): if the question relates to this hackathon project, link it to the project context from the `memory` skill.

## Writing Rules

- Plain words, short sentences; define a term the first time it appears.
- Prefer concrete examples and small numbers over abstract statements.
- Be accurate: simplify, but never state something false. Flag uncertainty.
- Match depth to the question; do not pad.

## Visuals

Pick the simplest visual that explains the idea:

- **Diagrams** (flows, pipelines, architectures, relationships): Mermaid code blocks in the answer; ASCII diagrams when Mermaid will not render.
- **Images** (plots, curves, comparisons, labelled illustrations): write a small Python script (matplotlib) or an SVG, render it to PNG/SVG, and show the result.
- **Videos / animations** (processes that change over time, e.g. training steps, robot motion, signal flow): use matplotlib `FuncAnimation` to produce a GIF or MP4.
- **Interactive pages**: for rich explanations, publish an HTML artifact with the text, diagrams and visuals together.

Visual rules:
- Save generated scripts and outputs to the session scratchpad directory, not the project folders, unless the user asks to keep them in the repo. Never write into raw data locations.
- Label every axis, node and arrow; add a one-line caption saying what to notice.
- Use only illustrative or real project data; mark illustrative data as such.
- Photorealistic images or real video cannot be generated directly; say so and use a diagram, plot or animation instead.
