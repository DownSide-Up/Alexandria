# Alexandria

**Open-source search infrastructure for the EU legislative record**

Alexandria is a self-hostable search engine for EU public policy documents. It ingests from public EU data sources, indexes documents with awareness of their legislative structure, and exposes the result as a documented REST/JSON API.

EU policy data is technically public but practically inaccessible. There is no open-source, self-hostable tool that aggregates the EU legislative record in a form that civil society organisations and developers can search without building it themselves. Alexandria addresses that gap.

---

## What it does

Alexandria pulls from public EU data sources — EUR-Lex/CELLAR, the European Parliament Open Data Portal, Parltrack — and from documents operators provide directly (FOI responses, grey literature, position papers). It understands the structure of EU legislative documents — articles, recitals, amendments, annexes — and indexes them accordingly. The API lets you search by meaning, by metadata, or both.

The initial document set covers EU tobacco and food systems policy, 2018–2026. The design extends to any EU policy domain.

**What Alexandria is not:**
- It does not include a frontend or user interface
- It does not host private or organisational data
- It does not generate text or answers
- It does not require any external SaaS dependency to run

---

## How it works

```
Source connectors             Ingest worker                   Query layer
─────────────────             ─────────────                   ───────────
CELLAR (SPARQL)      ───►     Parse (Formex XML,          ───► Qdrant
EP Open Data API     ───►       PDF fallback)             ───► (dense + sparse vectors)
Parltrack            ───►     Structure-aware chunking    ───►
Manual upload        ───►       (article / recital /      ───► REST/JSON API
(file + sidecar)               amendment / annex)             (FastAPI + OpenAPI)
                              Embed via BGE-M3
                              Store metadata (Postgres)
```

Two ingest modes — both first-class:

- **Automated (Mode A):** connectors pull from CELLAR/EP/Parltrack on a schedule. Metadata comes from the source and is applied automatically.
- **Manual upload (Mode B):** operator provides a file (PDF, DOCX, plain text) plus a metadata sidecar. This is the path for FOI-obtained documents, grey literature, and position papers that do not appear in public APIs.

**Stack:** Python · FastAPI · Qdrant · BGE-M3 · text-embeddings-inference · PostgreSQL · Docker  
**Licence:** AGPL-3.0 throughout. All dependencies are AGPL-compatible (MIT, Apache 2.0, BSD).  
**Deployment:** Docker Compose. Target spec: 4 vCPU, 16 GB RAM, 50 GB SSD

---

## Query API

The API is documented via auto-generated OpenAPI at `/docs`. Key endpoints:

```
GET  /v1/search           Semantic, hybrid, or metadata-filter search
GET  /v1/documents/{id}   Document metadata and chunk list
GET  /v1/chunks/{id}      Single chunk content and metadata
GET  /v1/documents        Paginated metadata-filter listing
GET  /v1/sources          Configured ingest sources and status
GET  /v1/stats            Corpus size and last-refresh times
GET  /openapi.json        OpenAPI 3.1 spec
```

Search supports three modes via `?mode=`:

- `hybrid` (default): dense + sparse vectors with score fusion — best for legislative text where exact terms ("Article 7", "TPD") matter alongside semantic meaning
- `semantic`: dense vector similarity only
- metadata filter: `?document_type=directive&sector=tobacco&date_from=2022-01-01`

Anonymous access is supported by default. Optional API keys enable per-key rate-limit accounting.

---

## Status

This repository is pre-release. 

The design is grounded in working prototypes built between 2024–2026:
- A tobacco amendments tracker grading 1,100+ EP amendments against civil society positions
- A legislative intelligence tool for food systems coalition use,
- A country-level nicotine legislation mapper across 37 European countries

These prototypes validated the data sources, document parsing approach, and search design. Alexandria is the open-source generalisation of that work.


## Data sources

| Source | Content | Access |
|---|---|---|
| EUR-Lex / CELLAR | EU legislation, preparatory acts, Commission documents | Public SPARQL + REST |
| EP Open Data Portal | MEP records, committee documents, adopted texts | Public REST API |
| Parltrack | MEP voting records, amendment tracking | Bulk JSON + scraper |
| Manual upload | FOI responses, grey literature, operator-curated documents | File + metadata sidecar |

Verified document count for the initial tobacco + food systems scope: approximately 2,000–8,000 CELLAR documents (confirmed via SPARQL query against the live endpoint, May 2026), producing an estimated 100,000–400,000 chunks.

---

## Self-hosting

The goal is a working instance in under 30 minutes from a clean Ubuntu box.

```bash
git clone https://github.com/DownSide-Up/Alexandria.git
cd Alexandria
cp .env.example .env        # edit to configure sources and credentials
docker compose up
```

The deployment runs five containers: `alexandria-api`, `alexandria-ingest-worker`, `qdrant`, `text-embeddings-inference`, and `postgres-metadata`. A minimal single-image path is also provided for smaller operator deployments.

---

## Roadmap

**v0.1 — initial document set (9-month scope)**
- [ ] CELLAR and EP Open Data connectors
- [ ] Manual upload pathway with metadata sidecar
- [ ] Formex XML structure-aware document parser
- [ ] BGE-M3 indexing pipeline via text-embeddings-inference
- [ ] Qdrant search index with hybrid (dense + sparse) support
- [ ] FastAPI query API with OpenAPI docs
- [ ] Docker Compose deployment
- [ ] Security review of public API surface
- [ ] Initial document set: EU tobacco and food systems policy 2018–2026

**Future work (not in scope for v0.1)**
- Layer 2 accessibility tooling for non-technical civil society users
- Private/organisational data layer
- Cross-organisation federated sharing

---

## Relation to DSU's broader work

Alexandria is part of [DownSideUp](https://downside-up.net)'s infrastructure work on civil society intelligence capacity. The argument: industry has always been able to afford the legal, intelligence, and political capacity to navigate EU legislative processes. Civil society organisations — the NGOs, advocacy groups, and watchdogs that hold them to account — cannot. Alexandria corrects one dimension of that asymmetry: making the raw public record equally searchable.

The broader DSU architecture (organisational-private layers, cross-org federation, the Dstil intelligence platform) is developed separately. This repository is scoped to the public commons layer only.

---

## Contributing

Development has not yet started. Contributing guidelines will be published alongside the first release. In the meantime, issues and discussion are open — particularly welcome are:
- Civil society organisations interested in early access or validation
- Developers with experience in EU legislative data formats (Formex, Akoma Ntoso)
- Organisations interested in self-hosting an instance

---

## Licence

[GNU Affero General Public Licence v3.0](LICENSE)

Alexandria is AGPL-licensed. Any modified version run as a network service must also be made available under the AGPL. All dependencies are AGPL-compatible.

---

*Built by [DownSideUp](https://downside-up.net) · Hamburg / Amsterdam / Brussels*
