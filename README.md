# Blood RNA-seq Atlas

Public human blood RNA-seq samples from SRA/ENA with all their BioSample metadata, their runs, and event labels
(what happened to the donor and when, relative to the blood draw). The dashboard is `index.html`; the data is a static
JSON API (`api/`) and bulk CSV files (`data/`). Agents: start at `llms.txt` or `api/index.json`.

## How it is made

1. **Samples.** ENA's table of every human RNA-seq run (library strategy RNA-Seq, source TRANSCRIPTOMIC), filtered to
   blood: whole blood, PBMC, blood immune cells, plasma/serum RNA, platelets and cord blood, with bone marrow as its own
   class. Cell lines, iPSC, organoids and xenografts are left out.
2. **Metadata.** Every sample's BioSample record (NCBI; ENA for the rest) and every study's ENA record (GEO-submitted
   projects carry the GEO summary).
3. **Labels.** Each sample's metadata, and each study's description, is read by a trained event classifier with a local
   LLM for the cases it is unsure about: 37 clinical events (infections, sepsis, vaccination, viral challenge, surgery,
   transplant, treatments, ...), their certainty, and their time from the blood draw.
4. **Quality.** Six tests without human annotation (see `api/summary.json` → `quality`): agreement with the structured
   disease/infection/vaccine fields, the same with those fields hidden, timing against explicit time points ("day 7
   post-vaccination"), healthy controls, in-vitro-stimulated samples, and classifier vs LLM agreement.

## Licence

Metadata as deposited in SRA/ENA/BioSample. Labels and harmonisation: CC BY 4.0. Samples marked `controlled` have public
metadata but their reads need dbGaP approval.
