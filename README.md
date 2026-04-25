# HGIS Canada — Census Subdivisions, 1851–1921

Static HTML site publishing one prose page per Canadian Census Subdivision per census year, with cross-year continuity, Wikidata grounding, neighbour links, and Schema.org structured data.

**Live site:** [jimclifford.ca/hgiscanada](https://jimclifford.ca/hgiscanada/)

## What this repo contains

- `index.html` — site homepage with coverage table and sample links.
- `places/on/<name>-<tcpuid>-<year>/index.html` — one page per Ontario CSD per census year (~6,500 pages today; more provinces to come).
- `sitemap.xml` — auto-generated; submit to Google Search Console after deploy.
- `robots.txt` — allow-all + sitemap pointer.
- `.nojekyll` — tells GitHub Pages to serve files raw, no Jekyll build step.

No Jekyll, no Liquid, no themes. Plain HTML with inline styles. Each page is self-contained and ~6 KB.

## How pages are generated

Pages are **derived output**, not authored by hand. The source-of-truth is the [Canada History Knowledge Graph](https://github.com/jburnford/Canada-History-Knowledge-Graph) repo, which hosts:

- A KuzuDB property graph as the editing layer
- The generator script: `scripts/generate_rag_pages.py`
- A CIDOC-CRM Turtle export for the LINCS LOD ecosystem

To rebuild this site:

```bash
# In the source repo
python3 scripts/generate_rag_pages.py --all
# Then sync rag_site/ → this repo
rsync -a --delete rag_site/ ~/hgiscanada/
cd ~/hgiscanada && git add -A && git commit -m "Refresh" && git push
```

## Why this exists

Default-mode chatbots (Gemini, Claude, ChatGPT) can't read Borealis ZIP files, can't query SPARQL endpoints, and can't open graph databases. They *can* read indexed prose pages with structured data. This site is the bridge between the structured knowledge graph and citizen question answering: ask a chatbot "what was the population of Westmeath, Ontario in 1871?" and an indexed page from this site is the citation it needs.

## Coverage

| Year | ON CSD pages |
|------|-------------:|
| 1851 | 406 |
| 1861 | 513 |
| 1871 | 589 |
| 1881 | 719 |
| 1891 | 796 |
| 1901 | 935 |
| 1911 | 1,025 |
| 1921 | 1,513 |
| **Total** | **6,496** |

Wikidata grounding: 2,196 pages currently link to a Wikidata QID; the rest are awaiting either disambiguation pipeline completion or a stale-cache refresh in the source repo.

## Identity model

Each page describes a **Place** (an enduring concept like "Westmeath") in a specific **Presence** (its 1871 census manifestation). Identity across years is established by spatial polygon overlap, not name match — so a township that was renamed or whose neighbours shifted still threads through to the right modern entity. Where a Place has been linked to Wikidata, that QID is treated as the canonical identifier; deep-time questions inherit Wikidata's depth (or its limits).

## License & citation

Census data is from the [Canadian Peoples / TCP project](https://borealisdata.ca/dataverse/canadiansubdivisions). The knowledge graph and these pages are released for academic and citizen use. Please cite both the TCP source and the [Canada History Knowledge Graph](https://github.com/jburnford/Canada-History-Knowledge-Graph) project.
