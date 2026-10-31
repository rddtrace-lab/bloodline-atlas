# How the labels are validated

The labels are made by machine (a trained event classifier, with a local LLM for the cases it is unsure about). No
labels were made by us by hand. Their accuracy is measured four independent ways; every number is on the dashboard and
in `api/summary.json` (`validation`, `biocheck`, `quality`).

## 1. Against a human-curated reference (Gemma)

[Gemma](https://gemma.msl.ubc.ca) curators annotate the experimental design of each GEO dataset by hand: for every
sample, its factor values (disease, treatment, time point, ...) as ontology terms. 6,425 atlas samples in 187
datasets carry such curation; those whose curated values name one of the atlas's events, or a control within a
dataset that does, form the reference.

- Mapping from curated terms to events (fixed in code before any label was compared): a value naming an event
  (sepsis; COVID-19, influenza, RSV, dengue ...; bacterial infection, tuberculosis, leprosy ...; vaccination;
  surgery; stroke / cerebral haemorrhage; trauma; ...) is a positive for it; a control value (reference subject role,
  control, healthy, uninfected, mock, placebo, baseline, pre-...) in the same dataset and factor is a negative.
  Pathogens in a treatment/agent factor are left out: they were put on cells in the dish.
- Time: curated time points ("1 d", "7 d", "24 h", "initial time point") in datasets with a timed event give the
  reference offset from the event to the draw.
- Metrics: sensitivity, specificity, precision, F1 and Cohen's kappa, pooled and per event; time-bucket agreement and
  error in days; the share of curated controls given any event. 95% confidence intervals come from a bootstrap over
  datasets, because samples of one dataset are not independent.
- Baseline: a plain keyword match on the same metadata, to show what the model adds over string matching.

## 2. Against the RNA itself (ARCHS4)

If a label is right, the blood's expression should show it. Using ARCHS4's uniformly processed gene counts, inside
each study (same lab and protocol), then pooled across studies by random-effects meta-analysis of AUCs:

- viral infection labels vs an interferon score (IFI27, IFI44L, ISG15, RSAD2, SIGLEC1, IFI44, MX1, OAS1);
- bacterial infection / sepsis labels vs a neutrophil score (CD177, MMP8, OLFM4, LCN2, HP, ARG1, CEACAM8);
- vaccination timing: interferon 1-3 days and plasmablasts (JCHAIN, MZB1, TXNDC5, TNFRSF17) 6-8 days after
  vaccination, each against samples drawn before it: the known kinetics appear only if the time labels are right;
- RNA-predicted sex (XIST vs Y-chromosome genes) against the sex field, a check of the metadata itself.

## 3. Internal consistency

Six rule-based tests on the structured BioSample fields (`quality`): agreement with disease/infection/vaccine fields
and with negated ones, the same with those fields hidden, timing against explicit time points ("day 7
post-vaccination"), healthy controls, in-vitro-stimulated samples, and classifier vs LLM agreement.

## 4. Known limits

- Curated references cover a minority of samples and of events; per-event numbers with few datasets have wide CIs.
- The RNA check shows that labelled groups differ as biology predicts; it does not score single samples.
- Single-cell studies are flagged (`single_cell`); their "samples" are often cells or pools.
