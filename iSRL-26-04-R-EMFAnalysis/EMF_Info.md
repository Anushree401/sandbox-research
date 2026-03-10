# EMF Scoring Model

This document explains the emf model in a simplified way to assist discussion about iSRL-26-0x-R-Metadata [(#6)](https://github.com/isrl-research/sandbox-research/discussions/6) and iSRL-26-04-D-Modelling [(#8)](https://github.com/isrl-research/sandbox-research/discussions/8) for contributors involved in data modelling design decisions. If you have a specific query about the logic behind scoring or nutrtional/ legal aspects, check the papers listed below and tag @isrl-research/research-core in relevant thread.

EMF is a scoring system that describes how far a food ingredient has drifted from its original biological source. Three scores — E, M, F — each capture a different dimension of that drift. The full model is described in the main paper and justified in the companion.

> The tagged_variants.csv is released as a work in progress dataset to enable discussions related to iSRL-26-04-D-Modelling. The labels may contain errors. Please treat it as sample data to decide the modelling needed instead of finalising.

---

## The Three Axes

### E — Anthropogenic Energy Score `[0, 1]`

What kind of process was applied? Lower scores = simple physical steps; higher scores = chemical synthesis.

```
processes = [sorting, washing, chilling, de-husking, milling, cold pressing, churning,
             pasteurization, clarification, fermentation, roasting, refining, fractionation,
             solvent extraction, interesterification, hydrogenation, acetylation,
             synthetic vanillin, synthetic flavors]
```

### M — Matter Score `[0, 1]`

How different is the ingredient's physical/chemical form from what it started as?

```
matter_states = [whole/fresh pieces, cut/sliced pieces, pulp/puree, coarse grits,
                 flour/fine powder, flakes, dense block, concentrate, powder, juice,
                 whey powder, skim/defatted meal, starch flour, oil, fat fraction,
                 protein concentrate, protein isolate, granules, extract/oleoresin,
                 oleoresin, emulsifier powder, essential oil, modified starch powder,
                 crystalline chemical]
```

### F — Functional Score `[0, 1]`

Is the ingredient's identity defined by its source or by what it does in a product? Low F = source-first identity. High F = function-first identity.

```
functional_classes = [base ingredient, taste profile / spice, lipid base, bulking agent,
                      humectant, firming agent, raising agent, flavouring agent, thickener,
                      stabiliser, gelling agent, sweetener, foaming agent, colour, emulsifier,
                      anticaking agent, acidity regulator, antioxidant, preservative,
                      antifoaming agent, sequestrant, bleaching agent, flour treatment agent,
                      carrier, propellant, packaging gas]
```

---

## N — Natural States

The botanical part of the source organism the ingredient came from. This is not a scored axis — it records which plant part (or animal/microbial equivalent) the ingredient originates from.

```
natural_states = [fruit, seed, bark, root, leaf, stem, flower, bud, resin, gum, latex,
                  rhizome, bulb, tuber, pod, hull, husk, peel, rind, pith, pulp, kernel,
                  nut, berry, grain, spore, shoot, tendril, stamen, pistil, sap, exudate]
```

---

## Using the CSV Files

Three companion CSVs ship with this model, one per scored axis:

| File | Columns | Rows |
|---|---|---|
| `e_scores.csv` | `tag, score` | 19 process tags |
| `m_scores.csv` | `tag, score` | 24 matter-state tags |
| `f_scores.csv` | `tag, score` | 26 functional-class tags (3 scores left blank — see below) |

**Merge on `tag`** to attach scores to any ingredient dataset that has been tagged with EMF labels:

```python
import pandas as pd

e = pd.read_csv('e_scores.csv')
m = pd.read_csv('m_scores.csv')
f = pd.read_csv('f_scores.csv')

# df must have columns named 'e_tag', 'm_tag', 'f_tag' (or rename as needed)
df = pd.merge(df, e.rename(columns={'tag': 'e_tag', 'score': 'E'}), on='e_tag', how='left')
df = pd.merge(df, m.rename(columns={'tag': 'm_tag', 'score': 'M'}), on='m_tag', how='left')
df = pd.merge(df, f.rename(columns={'tag': 'f_tag', 'score': 'F'}), on='f_tag', how='left')
```

**Blank scores in f_scores.csv:** `flavouring agent`, `sweetener`, and `colour` have no score assigned in the justification companion. Rows for these tags are present with an empty `score` field. Treat them as `NaN` in analysis until score assignment. Likewise for any tag prefixed with '?' like '?hydrolysis'.

## Flagging Disrepancy
If you see any disrepancy in the scores in the csv files here vs the justification report - Please create an issue / pr to fix it. For immediate analysis, consider the justification report as the ground truth and resolve conflicts according to the same.

---

## Papers

- **Main paper:** Identity, Transformation, and Function: A Tri-Axial Model for the Classification of Food Ingredient Identity
  <https://doi.org/10.5281/zenodo.18714527>

- **Justification companion:** Justification Companion to EMF-Scoring Model
  <https://doi.org/10.5281/zenodo.18713318>
