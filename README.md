# Supply Chain Mapper

An open-source pipeline and graph explorer that maps which brands source from which factories and mills, so investigators can quickly show who is responsible when workers are harmed.

> **Status: concept. Looking for contributors.**

## Why this exists

When a factory collapses, or when an owner closes a factory and disappears without paying wages or severance, the first question is which brands were buying from it. A brand that can be shown to have sourced from the factory can be pressed to make the workers whole.

Answering that question takes a long time. Supplier lists are published as PDFs and spreadsheets in many different formats, the same factory's name is spelled a dozen ways, and production is often subcontracted to factories that never appear on any list. Investigators end up rebuilding the same picture by hand, under time pressure, for every case.

## Who it's for

- Researchers at labor rights organizations, such as the Clean Clothes Campaign and the Worker Rights Consortium
- Investigative journalists
- Unions and worker advocates

This project is independent and is not affiliated with or endorsed by any of the organizations named here.

## Core features

**Automated ingestion.** Python scrapers watch for and ingest brand supplier lists, which many brands publish as PDF or Excel files under the Transparency Pledge. They also ingest public customs records such as US import bills of lading.

**Entity resolution.** The pipeline merges records that refer to the same real-world entity. For example, it should recognize that "Dhaka Garments Ltd." and "Dhaka Garment Limited" at the same address are one factory. It uses name normalization, address matching, and shared identifiers. Uncertain matches go to a human review queue instead of being merged automatically.

**Graph explorer.** Search for a factory and see every brand linked to it, the parent company behind each brand, the sources for each link, and a timeline of each relationship.

**Discrepancy flags.** The explorer highlights where sources disagree. For example, a brand says it stopped sourcing from a factory in 2024, but customs records show shipments in 2025.

**Provenance on everything.** Every link in the graph points back to the document it came from, along with the date it was retrieved. A finding is only as useful as the evidence behind it.

## Data model (draft)

```
(Brand)-[:OWNED_BY]->(Company)
(Brand)-[:SOURCES_FROM {start, end, source}]->(Facility)
(Facility)-[:SUBCONTRACTS_TO {start, end, source}]->(Facility)
(Facility)-[:SUPPLIED_BY {start, end, source}]->(Facility)   // e.g. a tier-1 factory supplied by a tier-2 mill
(Shipment {date, quantity, source})-[:SHIPPED_FROM]->(Facility)
(Shipment)-[:CONSIGNED_TO]->(Brand)
```

Each `source` points to a stored copy of the original document, with its URL, retrieval date, and file hash.

## Data sources to evaluate

- **Brand supplier lists** published under the Transparency Pledge or voluntarily.
- **[Open Supply Hub](https://opensupplyhub.org/)**: an open, searchable database of production facilities with stable facility IDs. We should use its IDs as canonical identifiers and contribute data back instead of duplicating its work.
- **US import bills of lading**, available from US Customs and Border Protection and from commercial data providers. Access and redistribution terms vary.
- **Company registries**, such as [OpenCorporates](https://opencorporates.com/), for parent-company ownership.

Each source has its own terms of use. Scrapers must respect them, and we will record the license of every dataset we ingest.

## Proposed architecture

```
Scrapers (Scrapy)
  → raw document store (original files, hashed and dated)
  → parsers (PDF / Excel / CSV → tables, with pandas)
  → entity resolution (+ human review queue)
  → graph database (Neo4j or ArangoDB)
  → API
  → web explorer (React + Cytoscape.js or D3.js)
```

## Suggested tech stack

Everything below is a starting point and open for discussion in issues.

| Layer | Options |
| --- | --- |
| Ingestion | Python, Scrapy, pandas; a PDF table extractor such as Camelot or pdfplumber |
| Entity resolution | Python; rule-based matching first, with a library such as Splink or dedupe evaluated later |
| Graph database | Neo4j or ArangoDB |
| API | Python (FastAPI) or Node.js |
| Frontend | React with Cytoscape.js or D3.js |

## Responsible use

People will use this tool to make public claims about real companies. Getting a claim wrong can damage workers' cases and expose users to legal risk. To guard against that:

- Every link in the graph cites a source. A link without a source doesn't go in the graph.
- Matches carry a confidence score, and low-confidence matches need human sign-off.
- Exports and screenshots include their sources and retrieval dates.
- The graph holds no personal information about individual workers.

## Roadmap

1. Ingest a handful of brand supplier lists into a common schema.
2. Match facilities against Open Supply Hub IDs, with a review queue for uncertain matches.
3. Set up the graph database and a basic search-and-explore UI.
4. Ingest customs data and add the timeline and discrepancy views.
5. Monitor supplier lists on a schedule and send alerts when they change.

## Contributing

Help is especially welcome from:

- Python and data engineers (extracting tables from PDFs is a hard, unglamorous problem)
- People with experience in entity resolution and record linkage
- Graph database and frontend visualization developers
- Researchers and journalists who know where the data lives and how it's used in real cases

Please open an issue to discuss larger changes before starting work on them.

## License

[MIT](LICENSE)
