---
title: "llmde"
pitch: "AI-assisted data extraction for systematic reviews, with every value traceable to the page and quote it came from."
icon: wand-magic-sparkles
weight: 40
meta:
  - PhD thesis, University of Geneva, 2026
  - With [Mathieu Scheltienne](https://mathieu.scheltienne.net), package engineering
  - "Role: project lead, extraction method and prompt design"
  - Open source, MIT licence
skills: ["LLM workflow design", "Evidence synthesis", "Reproducible research"]
links:
  - label: View on GitHub
    url: https://github.com/naimagradi/llmde
note: "The model reached 97% precision against the manual extraction."
---

**A language-model extraction pipeline for a systematic review and meta-analysis, built so that every value carries
its page and quotation and a human reviewer keeps the last word.**

## What it does

Extracting study data from 39 trials by hand is slow and inconsistent between readers. llmde hands the mechanical
part to a language model under strict rules: extract exactly, never interpret, return an explicit null for anything
unreported, and cite the page and quote behind every value.

{{< figure src="images/extraction-architectures.png" alt="Three architectures for language-model data extraction: one-shot as implemented, agentic, agentic with a human gate" caption="Figure 2 of the thesis. Panel (a) is the design implemented: one call per article and prompt, every value with its page and quote, checked against a manual extraction. Panels (b) and (c) are the agentic alternatives, the second with a human gate, proposed but not built." >}}

## How the prompts were built

Each prompt was drafted, run on two or three deliberately varied papers in parallel with a human reader, and revised
wherever the two disagreed. Mismatches were diagnosed before anything was changed, since most came from a definition
with two readings or a reporting variant with no rule.

{{< figure src="images/calibration-loop.en.svg" alt="Prompt calibration loop: model and human read the same paper, mismatches are diagnosed and the prompt revised until they agree" caption="The loop ran on two or three varied papers until model and human agreed; only then was the prompt frozen and run on the whole corpus." >}}

## What came out

- **Duplicate extraction.** 39 trials for the review and 19 for the meta-analysis, once by hand and once by the
  model (Claude Opus 4.6), reconciled by discussion, with 97% precision for the model against the manual
  extraction.
- **Open-source package.** llmde on GitHub, MIT licence: five extraction prompts, one system instruction, Claude and
  Gemini backends, a batch CLI that resumes interrupted runs.
- **Written method.** Chapter 5 of the thesis: the single-call design next to two agentic alternatives, with a case
  for keeping a human gate.

## My role

I led the project: I designed the extraction method and co-wrote the prompts and schemas, and
[Mathieu Scheltienne](https://mathieu.scheltienne.net) implemented them as the package. I ran the model side of the
duplicate extraction and reconciled it with the manual side.

## What this shows

- LLM workflows designed with verification built in: evidence trail, explicit nulls, structural checks, pinned
  model version.
- Evidence-synthesis method: duplicate extraction and calibration on a pilot sample, as review guidance recommends.
- Specifying a method precisely enough for an engineer to turn it into a reusable tool.
