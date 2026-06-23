# Notice and Attribution

This repository is a public learning, secondary-development and interview-reference version of dream-rag. It may incorporate code, scaffolding, examples, or module structures derived from upstream open-source projects, engineering templates, or tutorial materials.

## Upstream Projects and Dependencies

This project builds on the following open-source frameworks and services:

| Project | License | Usage |
|---------|---------|-------|
| Spring Boot | Apache 2.0 | Application framework |
| Vue 3 | MIT | Frontend framework |
| Naive UI | MIT | UI component library |
| Apache Kafka | Apache 2.0 | Message queue / async processing |
| Elasticsearch | Elastic License 2.0 / SSPL | Search engine |
| Redis | BSD-3-Clause | Caching / session state |
| MinIO | AGPLv3 | Object storage |
| MySQL | GPLv2 (server) | Relational database |
| Apache Tika | Apache 2.0 | Document parsing |
| Apache PDFBox | Apache 2.0 | PDF processing |

## Attribution Policy

- Do not remove upstream author comments, copyright headers, license files, or notices.
- If a previously unlisted upstream project is identified, add its name, repository link, and license here.
- Template, scaffold, and admin-panel code (e.g., Soybean Admin patterns) should not be presented as original work.

## Historical Notes

The project was previously developed under internal naming conventions including "pai-smart" and related variants. Some file naming, configuration keys, and package references may retain traces of these earlier stages. These are being progressively cleaned up and do not affect functionality.

## Scope of Original Contribution

The core value demonstrated in this repository lies in:

- RAG document processing pipeline (upload → parse → chunk → vectorize)
- Kafka-based async processing with retry and dead-letter queue
- Elasticsearch hybrid search (BM25 + vector)
- WebSocket streaming chat with WebFlux SSE consumption
- Redis-based generation state management and reconnection support
- Multi-tenant authorization with organization tag filtering
- Docker Compose orchestration with health checks

Third-party infrastructure, generic admin CRUD, billing/token-recharge modules, and UI scaffolding are supporting components and should be evaluated as such.
