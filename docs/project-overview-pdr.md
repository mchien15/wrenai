# WrenAI - Project Overview & Product Development Requirements

## Product Overview

WrenAI is an open-source GenBI (Generative Business Intelligence) agent that converts natural language questions into SQL queries and data visualizations. It uses a semantic layer (MDL - Metadata Definition Language) to guide LLM-powered text-to-SQL generation via retrieval-augmented generation (RAG).

**Key Value Propositions:**
- **Natural Language to SQL**: Ask questions in plain language, get accurate SQL queries
- **Semantic Layer Control**: MDL models encode business logic, ensuring accurate and governed LLM outputs
- **Multi-LLM Support**: OpenAI, Azure, Anthropic, Google, DeepSeek, Groq, Ollama, Databricks
- **13+ Data Sources**: Athena, Redshift, BigQuery, DuckDB, Databricks, PostgreSQL, MySQL, SQL Server, ClickHouse, Oracle, Trino, Snowflake
- **Chart Generation**: Automatic text-to-chart with Vega/Vega-Lite
- **API Integration**: Embed query and chart generation in custom applications

## Monorepo Structure

```
wrenai/
├── wren-ui/              # Next.js 14 frontend + Apollo GraphQL backend
├── wren-ai-service/      # Python FastAPI AI/LLM service with RAG pipelines
├── wren-launcher/        # Go CLI deployment tool
├── wren-engine/          # SQL engine (git submodule, external)
├── wren-mdl/             # MDL JSON schema definitions
├── docker/               # Docker Compose configs for local development
└── deployment/           # Kubernetes/Kustomize manifests for production
```

## Service Architectures

### wren-ui (TypeScript)
- **Framework**: Next.js 14 with App Router
- **GraphQL**: Apollo Server embedded in Next.js API routes
- **Database**: SQLite (dev) / PostgreSQL (prod) via Knex.js
- **UI Library**: Ant Design + styled-components
- **Visualization**: react-grid-layout, reactflow, vega/vega-lite
- **Testing**: Jest (unit), Playwright (E2E)
- **LOC**: ~61K TypeScript files

**Internal Architecture** (Domain-Driven Design):
```
src/apollo/server/
├── resolvers/          # GraphQL query/mutation handlers
├── services/           # Business logic layer
├── repositories/       # Data access layer (Knex)
├── adaptors/           # External service adapters (AI service, engine)
└── types/              # TypeScript types & interfaces
```

**Path Aliases**:
- `@/*` → `./src/*`
- `@server/*` → `./src/apollo/server/*`

### wren-ai-service (Python)
- **Framework**: FastAPI
- **RAG Framework**: Haystack AI + Hamilton dataflow
- **Vector DB**: Qdrant
- **LLM Gateway**: LiteLLM (multi-provider support)
- **Testing**: pytest + locust (load testing)
- **LOC**: ~15.4K Python files

**Pipeline Architecture**:
```
src/pipelines/
├── indexing/           # MDL, questions, SQL pairs → Qdrant
├── retrieval/          # Semantic search from Qdrant
├── generation/         # SQL, chart, intent generation
├── ask/                # Orchestrates retrieval + generation
└── ask_details/        # SQL breakdown and explanation
```

**Configuration**: Multi-document YAML (`config.yaml`) with sections for LLM, embedder, engine, document_store, pipeline, settings. Environment via `.env.dev`.

### wren-launcher (Go)
- **Purpose**: CLI for launching services and dbt-to-MDL conversion
- **Key Libraries**: Docker Compose API, pterm, promptui
- **LOC**: ~4.7K Go files (17 .go files)

## Product Development Requirements (PDR)

### Functional Requirements

#### FR1: Natural Language Query Processing
- Accept natural language questions in any language
- Parse user intent and extract relevant entities
- Map questions to MDL schema elements
- Generate syntactically correct SQL queries
- Support complex queries with joins, aggregations, filters

#### FR2: Semantic Layer Management
- Define MDL models with tables, columns, relationships
- Specify metrics and calculated fields
- Manage column descriptions and business definitions
- Support historical Q&A indexing
- Store SQL pair examples for few-shot learning

#### FR3: Data Source Integration
- Connect to 13+ supported data sources
- Validate connections and credentials
- Support multiple data sources per project
- Handle source-specific SQL dialect differences

#### FR4: Query Execution & Visualization
- Execute generated SQL against data sources
- Return query results in tabular format
- Generate appropriate chart types (bar, line, pie, etc.)
- Support chart customization (colors, labels, axes)
- Export results (CSV, JSON, images)

#### FR5: LLM Provider Flexibility
- Support 9+ LLM providers via LiteLLM
- Configure different models for different pipelines
- Handle provider-specific API authentication
- Fallback and retry logic for API failures

#### FR6: RAG Pipeline Orchestration
- Index MDL schema, questions, SQL pairs to vector store
- Retrieve relevant context via semantic search
- Use retrieved context for prompt engineering
- Iterative SQL correction based on validation feedback

### Non-Functional Requirements

#### NFR1: Performance
- Query response time < 10 seconds for typical questions
- Indexing latency < 30 seconds for medium MDL schemas
- Support 100+ concurrent users in production
- UI page load time < 2 seconds

#### NFR2: Accuracy
- SQL execution success rate > 85% for supported queries
- Semantic search recall > 80% for relevant context
- Chart type selection accuracy > 75% for data patterns

#### NFR3: Security
- Encrypt data source credentials at rest
- Support API key authentication for LLM providers
- SQL injection prevention via parameterized queries
- Role-based access control for multi-tenant deployments

#### NFR4: Scalability
- Horizontal scaling for all services (stateless design)
- Vector store scaling for large MDL schemas
- Database connection pooling for concurrent requests
- CDN caching for static assets

#### NFR5: Maintainability
- Domain-driven architecture for clear separation of concerns
- Comprehensive test coverage (>70% for critical paths)
- Structured logging for debugging
- Health check endpoints for monitoring

#### NFR6: Usability
- Intuitive UI for non-technical users
- Clear error messages and suggestions
- Interactive SQL editor for manual refinement
- Onboarding flow for first-time users

### Technical Constraints

#### TC1: Technology Stack
- Frontend: Next.js 14, TypeScript, Apollo GraphQL, Ant Design
- Backend: Python 3.12, FastAPI, Poetry, Just
- CLI: Go 1.18+, Make
- Database: SQLite (dev), PostgreSQL (prod)
- Vector DB: Qdrant

#### TC2: Deployment
- Docker Compose for local development
- Kubernetes for production deployment
- Docker images published to ghcr.io/canner/

#### TC3: Code Quality
- TypeScript strict mode with tsc --noEmit
- ESLint for linting
- Pre-commit hooks (AI service)
- Conventional commits format

#### TC4: API Design
- GraphQL for UI-backend communication
- REST for AI service endpoints
- OpenAPI/Swagger documentation

## Success Metrics

- **User Adoption**: 100+ GitHub stars, 10+ active contributors
- **Query Success Rate**: >85% of generated SQL execute successfully
- **Response Time**: P95 < 10s for end-to-end query processing
- **Data Source Coverage**: 13+ production connectors
- **LLM Flexibility**: 9+ provider integrations
- **Community Engagement**: 1.3k+ Discord members, active GitHub issues/PRs

## Dependencies

### External Services
- **LLM Providers**: OpenAI, Azure, Anthropic, Google, DeepSeek, Groq, Ollama, Databricks
- **Vector Database**: Qdrant
- **Data Sources**: Various database/cloud warehouse APIs

### Internal Modules
- **wren-engine**: SQL validation and execution (submodule)
- **wren-mdl**: JSON schema for MDL validation

## Release Strategy

- **Versioning**: Semantic versioning for all services
- **Release Cadence**: Monthly minor releases, weekly patch releases
- **Backward Compatibility**: Maintain for at least 2 major versions
- **Migration Guides**: Provided for breaking changes

## Open Questions

1. How to handle multi-tenant data isolation in production deployments?
2. What is the maximum MDL schema size supported before performance degrades?
3. Should we add streaming responses for long-running queries?
4. How to implement caching for repeated questions?
5. What metrics should be tracked for production monitoring?

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
