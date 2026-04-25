# HGIS Canada — Census Subdivisions, 1851–1921

Static HTML site publishing one prose page per Canadian Census Subdivision per census year, plus an aggregate page per persistent place. Includes the **full** census record (every variable recorded), Wikidata grounding where available, neighbour links, cross-year continuity (including non-identical boundary overlaps), and Schema.org structured data.

**Live site:** [jimclifford.ca/hgiscanada](https://jimclifford.ca/hgiscanada/)

## Coverage

All twelve Canadian provinces and territories: Ontario, Quebec, Nova Scotia, New Brunswick, Prince Edward Island, British Columbia, Alberta, Saskatchewan, Manitoba, Yukon, Northwest Territories, Newfoundland and Labrador.

Generated from the v2 KuzuDB knowledge graph (~13K persistent places, ~22K presences, ~1.56M census measurements).

## What this repo contains

- `index.html` — site homepage with coverage table and sample links.
- `places/<prov>/<name>-<tcpuid>-<year>/index.html` — one page per CSD per census year (~22K pages).
- `places/<prov>/<name>-<place-id>/index.html` — one page per persistent Place aggregating all years (~13K pages).
- `sitemap.xml` — auto-generated; submit to Google Search Console after deploy.
- `robots.txt` — allow-all + sitemap pointer.
- `.nojekyll` — tells GitHub Pages to serve files raw, no Jekyll build step.

No Jekyll, no Liquid, no themes. Plain HTML with inline styles. Each year-page contains the **full census record** for that subdivision (every variable recorded, grouped into collapsible category tables by the e55 type catalog: Population, Age, Ethnic origin, Religion, Buildings, Agriculture, Manufacturing, Fisheries, Deaths, Other).

## Page contents

**Year-specific pages** (e.g. Westmeath 1871, Trois-Rivières 1881, Lunenburg 1891):
- Population summary, sex split, density (where recorded)
- Cross-year population trajectory linking the same place across all 8 censuses via spatial polygon overlap
- **Boundary continuity** section linking adjacent-year polygons that overlap but didn't make the SAME_AS threshold (e.g. when a township split into a town + rural fragment, both fragments are linked from the predecessor's page)
- Neighbouring CSDs in that year (internally linked, including cross-province borders)
- **Full census record**: every variable recorded for this subdivision in this year, grouped into collapsible tables by category
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

- A KuzuDB property graph as the editing layer (Place, Presence, Name, CensusVariable, Measurement nodes; OBSERVED_IN, BORDERS, CONTINUES_AS, OVERLAPS_TEMPORALLY, SPLIT_FROM, MERGED_INTO, MEASURED_AT, OF_VARIABLE, PART_OF_COUNTY edges)
- The generator script: `scripts/generate_rag_pages.py`
- A CIDOC-CRM Turtle export for the LINCS LOD ecosystem

To rebuild this site:

```bash
# In the source repo
python3 scripts/export_kuzu_pilot.py        # Rebuild Kuzu (all of Canada)
python3 scripts/generate_rag_pages.py --all # Regenerate HTML
# Sync output → this repo
rsync -a --delete --exclude '.git' --exclude README.md \
  ~/Canada-History-Knowledge-Graph/rag_site/ ~/hgiscanada/
cd ~/hgiscanada && git add -A && git commit -m "Refresh: <what changed>" && git push
```

## Why this exists

Default-mode chatbots (Gemini, Claude, ChatGPT) can't read Borealis ZIP files, can't query SPARQL endpoints, and can't open graph databases. They *can* read indexed prose pages with structured data. This site is the bridge between the structured knowledge graph and citizen question answering: ask a chatbot "what was the population of Trois-Rivières in 1881?" or "how many acres of barley did Alfred grow in 1871?" or "what townships bordered Lunenburg, Nova Scotia in 1891?" and an indexed page from this site is the citation it needs.

## Identity model

Each year-page describes a **Place** (an enduring concept) in a specific **Presence** (its census-year manifestation). Identity across years is established by spatial polygon overlap, not name match — so a township that was renamed or whose neighbours shifted still threads through to the right modern entity. Where a Place has been linked to Wikidata, that QID is treated as the canonical identifier; deep-time questions inherit Wikidata's depth (or its limits).

Where polygon boundaries shifted enough that the strict SAME_AS chain broke (IoU < 0.98), we still surface the relationship via `OVERLAPS_TEMPORALLY` edges — `CONTAINS`, `WITHIN`, and `OVERLAPS` overlap types render as a "Boundary continuity" section linking adjacent-year predecessors and successors even when they're tracked as different persistent places.

The per-Place index page (e.g. `/places/on/westmeath-on142032/`) is the natural landing page for "tell me about Westmeath" queries — deliberately above the year-specific detail pages in the URL hierarchy.

## License & citation

Census data is from the [Canadian Peoples / TCP project](https://borealisdata.ca/dataverse/canadiansubdivisions). The knowledge graph and these pages are released for academic and citizen use. Please cite both the TCP source and the [Canada History Knowledge Graph](https://github.com/jburnford/Canada-History-Knowledge-Graph) project.
