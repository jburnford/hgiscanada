# HGIS Canada — Census Subdivisions, 1851–1921

Static HTML site publishing one prose page per Canadian Census Subdivision per census year, plus an aggregate page per persistent place. Includes the **full** census record (all variables recorded), Wikidata grounding, neighbour links, cross-year continuity, and Schema.org structured data.

**Live site:** [jimclifford.ca/hgiscanada](https://jimclifford.ca/hgiscanada/)

## What this repo contains

- `index.html` — site homepage with coverage table and sample links.
- `places/on/<name>-<tcpuid>-<year>/index.html` — one page per Ontario CSD per census year (~6,500 pages).
- `places/on/<name>-<place-id>/index.html` — one page per persistent Place aggregating all years (~3,700 pages).
- `sitemap.xml` — auto-generated; submit to Google Search Console after deploy.
- `robots.txt` — allow-all + sitemap pointer.
- `.nojekyll` — tells GitHub Pages to serve files raw, no Jekyll build step.

No Jekyll, no Liquid, no themes. Plain HTML with inline styles. Each year-page is ~9 KB and contains the **full census record** for that subdivision (every variable recorded, grouped into collapsible category tables).

## Page contents

**Year-specific pages** (e.g. Westmeath 1871):
- Population summary, sex split, density (where recorded)
- Cross-year population trajectory linking the same place across all 8 censuses via spatial polygon overlap
- Neighbouring CSDs in that year (internally linked)
- **Full census record**: every variable recorded for this subdivision in this year, grouped into collapsible tables by category — Population & families, Age structure, Ethnic origin, Religion, Buildings & housing, Agriculture, Manufacturing & industry, Fisheries, Deaths & mortality, Other recorded variables
- Wikidata QID and persistent place ID
- Schema.org JSON-LD for search engines and citation-grounded RAG
- Source citation back to the Borealis dataset
- Footer link up to the per-Place index page

**Per-Place index pages** (e.g. Westmeath 1851–1921):
- Population trajectory across all years with links to each year's detail page
- Boundary lineage: predecessors (split from earlier places) and successors (merged into later places)
- Wikidata grounding and persistent place identifier
- Schema.org Place with `temporalCoverage`

## How pages are generated

Pages are **derived output**, not authored by hand. The source-of-truth is the [Canada History Knowledge Graph](https://github.com/jburnford/Canada-History-Knowledge-Graph) repo, which hosts:

- A KuzuDB property graph as the editing layer (Place, Presence, Name, CensusVariable, Measurement nodes)
- The generator script: `scripts/generate_rag_pages.py`
- A CIDOC-CRM Turtle export for the LINCS LOD ecosystem

To rebuild this site:

```bash
# In the source repo
python3 scripts/export_kuzu_pilot.py --province ON   # Rebuild Kuzu DB
python3 scripts/generate_rag_pages.py --all          # Regenerate HTML
# Sync output → this repo
rsync -a --delete --exclude '.git' --exclude README.md \
  ~/Canada-History-Knowledge-Graph/rag_site/ ~/hgiscanada/
cd ~/hgiscanada && git add -A && git commit -m "Refresh: <what changed>" && git push
```

## Why this exists

Default-mode chatbots (Gemini, Claude, ChatGPT) can't read Borealis ZIP files, can't query SPARQL endpoints, and can't open graph databases. They *can* read indexed prose pages with structured data. This site is the bridge between the structured knowledge graph and citizen question answering: ask a chatbot "what was the population of Westmeath, Ontario in 1871?" or "how many acres of barley did Alfred grow in 1871?" and an indexed page from this site is the citation it needs.

## Coverage (Ontario)

| Year | CSD pages | Total measurements |
|------|-------------:|-------------:|
| 1851 | 406 | varies |
| 1861 | 513 | |
| 1871 | 589 | |
| 1881 | 719 | |
| 1891 | 796 | |
| 1901 | 935 | |
| 1911 | 1,025 | |
| 1921 | 1,513 | |
| **Year-pages total** | **6,496** | **584,299** |
| **Place-index pages** | **3,657** | (aggregate) |

Wikidata grounding: 2,196 of 6,496 year-pages currently link to a QID, covering 1,047 of 3,657 persistent places. The other 2,610 places are pending pipeline completion or stale-cache refresh in the source repo. (When a place has no Wikidata equivalent — confirmed via MCP verification — it gets a permanent minted URI per LINCS conventions.)

## Identity model

Each year-page describes a **Place** (an enduring concept like "Westmeath") in a specific **Presence** (its 1871 census manifestation). Identity across years is established by spatial polygon overlap, not name match — so a township that was renamed or whose neighbours shifted still threads through to the right modern entity. Where a Place has been linked to Wikidata, that QID is treated as the canonical identifier; deep-time questions inherit Wikidata's depth (or its limits).

The per-Place index page (e.g. `/places/on/westmeath-on142032/`) is the natural landing page for "tell me about Westmeath" queries — it's deliberately above the year-specific detail pages in the URL hierarchy.

## License & citation

Census data is from the [Canadian Peoples / TCP project](https://borealisdata.ca/dataverse/canadiansubdivisions). The knowledge graph and these pages are released for academic and citizen use. Please cite both the TCP source and the [Canada History Knowledge Graph](https://github.com/jburnford/Canada-History-Knowledge-Graph) project.
