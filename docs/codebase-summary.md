# WrenAI - Codebase Summary

## Overview

WrenAI is a monorepo containing three main services: a Next.js frontend with embedded GraphQL backend, a Python FastAPI AI service, and a Go CLI tool. The codebase follows domain-driven design principles with clear separation of concerns.

## Repository Statistics

| Service | Language | Files | LOC | Framework | Purpose |
|---------|----------|-------|-----|-----------|---------|
| wren-ui | TypeScript | 400+ | ~61K | Next.js 14, Apollo GraphQL | Frontend + GraphQL API |
| wren-ai-service | Python | 93+ | ~15.4K | FastAPI, Haystack AI | RAG pipelines & LLM orchestration |
| wren-launcher | Go | 17 | ~4.7K | Docker Compose API | CLI deployment tool |
| wren-mdl | JSON Schema | 1 | 471 | - | MDL manifest definition |
| docker | YAML | ~10 | ~1K | Docker Compose | Local development stack |
| deployment | YAML | ~8 | 442 | Kubernetes + Kustomize | Production manifests |

**Total**: ~83K lines of code across all services

## Service Breakdown

### wren-ui (TypeScript/Next.js)

**Structure**:
```
wren-ui/
├── src/
│   ├── apollo/
│   │   ├── client/           # GraphQL client operations
│   │   └── server/           # GraphQL server (embedded in Next.js)
│   │       ├── resolvers/    # GraphQL query/mutation handlers
│   │       ├── services/     # Business logic layer
│   │       ├── repositories/ # Data access (Knex)
│   │       ├── adaptors/     # External service adapters
│   │       ├── models/       # Database models
│   │       ├── types/        # TypeScript types
│   │       └── utils/        # Utilities
│   ├── components/           # React components
│   │   ├── home/             # Home page components
│   │   ├── setup/            # Data source setup
│   │   ├── modeling/         # MDL modeling UI
│   │   ├── knowledge/        # Knowledge base management
│   │   └── common/           # Shared components
│   ├── pages/                # Next.js page routes
│   └── lib/                  # Utility libraries
├── knex/                     # Database migrations
├── tests/                    # Jest & Playwright tests
└── package.json              # Yarn 4.5.3 dependencies
```

**Key Technologies**:
- **UI Framework**: Next.js 14 (App Router), React 18
- **GraphQL**: Apollo Server (embedded), Apollo Client
- **Database**: Knex.js with SQLite (dev) or PostgreSQL (prod)
- **UI Components**: Ant Design, styled-components
- **Visualization**: react-grid-layout, reactflow, vega, vega-lite
- **Testing**: Jest (unit), Playwright (E2E)
- **Build**: yarn, TypeScript strict mode

**GraphQL Schema**:
- 24+ resolvers (asking, model, project, dashboard, etc.)
- Domain-driven structure: resolvers → services → repositories → adaptors
- Scalar types for Date, JSON, etc.

**Database**:
- 44 Knex migrations
- Tables: projects, data_sources, models, questions, dashboards, etc.
- Connection pooling via Knex

**Key Files**:
- `src/apollo/server/schema.ts` (26K LOC) - GraphQL schema definitions
- `src/apollo/server/resolvers.ts` - Main resolver mapping
- `src/apollo/server/dataSource.ts` (14K LOC) - Data source management

### wren-ai-service (Python/FastAPI)

**Structure**:
```
wren-ai-service/
├── src/
│   ├── pipelines/            # RAG pipeline implementations
│   │   ├── indexing/         # Index MDL, questions, SQL pairs → Qdrant
│   │   ├── retrieval/        # Semantic search from Qdrant
│   │   ├── generation/       # SQL, chart, intent generation
│   │   ├── ask/              # Main text-to-SQL pipeline
│   │   └── ask_details/      # SQL explanation pipeline
│   ├── web/
│   │   └── v1/
│   │       ├── services/     # Service layer
│   │       └── routers/      # FastAPI route handlers
│   ├── core/                 # Base abstractions
│   ├── config.py             # Pydantic settings
│   └── globals.py            # Service container wiring
├── tests/
│   ├── pytest/               # Unit tests
│   └── usecases/             # Integration tests
├── config.yaml               # Multi-document config
├── .env.dev                  # Environment variables
└── pyproject.toml            # Poetry dependencies
```

**Key Technologies**:
- **Framework**: FastAPI
- **RAG**: Haystack AI framework
- **Dataflow**: Hamilton (DAG orchestration)
- **Vector DB**: Qdrant
- **LLM Gateway**: LiteLLM (9+ providers)
- **Testing**: pytest, locust (load testing)
- **Config**: Pydantic Settings, multi-document YAML

**Pipeline Architecture**:

1. **Indexing Pipelines** (`pipelines/indexing/`):
   - MDL schema indexing (models, columns, relationships)
   - Historical questions indexing
   - SQL pair indexing (few-shot examples)
   - Output: Vector embeddings in Qdrant

2. **Retrieval Pipelines** (`pipelines/retrieval/`):
   - Semantic search for relevant schema context
   - Question similarity search
   - SQL pair retrieval

3. **Generation Pipelines** (`pipelines/generation/`):
   - SQL generation via LLM
   - Chart type selection and generation
   - Intent classification
   - Data profiling

4. **Ask Pipeline** (`pipelines/ask/`):
   - Orchestrates retrieval → generation → validation
   - Iterative SQL correction (up to max_sql_correction_retries)
   - Final response formatting

5. **Ask Details Pipeline** (`pipelines/ask_details/`):
   - SQL breakdown and explanation
   - Step-by-step reasoning

**Configuration**:
- `config.yaml`: 6 sections (llm, embedder, engine, document_store, pipeline, settings)
- Settings load order: defaults → env vars → .env.dev → config.yaml
- Pre-commit hooks: `poetry run pre-commit install`

**API Endpoints** (24+):
- `POST /v1/ask` - Main text-to-SQL endpoint
- `POST /v1/ask_details` - SQL explanation
- `POST /v1/semantics/preparation` - MDL indexing
- `POST /v1/generate_chart` - Chart generation
- `POST /v1/sql_pairs` - SQL pair management

### wren-launcher (Go)

**Structure**:
```
wren-launcher/
├── cmd/
│   └── root.go              # CLI entry point
├── pkg/
│   ├── docker/              # Docker Compose API wrapper
│   ├── promptui/            # Interactive prompts
│   └── dbt/                 # dbt-to-MDL conversion
├── main.go                  # Application entry
├── go.mod                   # Go modules
└── Makefile                 # Build commands
```

**Key Technologies**:
- **CLI**: pterm (terminal UI), promptui (interactive prompts)
- **Docker**: Docker Compose API (go docker/compose)
- **Build**: go build, Make (cross-compilation)

**Features**:
- Launch all services via Docker Compose
- Interactive configuration setup
- dbt project to MDL conversion
- Health check monitoring

### docker (Development Stack)

**Services** (6 containers):
1. **bootstrap** - Initialization and configuration
2. **wren-engine** - SQL validation/execution (:8080)
3. **ibis-server** - SQL abstraction for data sources (:8000)
4. **wren-ai-service** - AI/LLM service (:5556)
5. **qdrant** - Vector database (:6333)
6. **wren-ui** - Frontend + GraphQL (:3000)

**Configuration**:
- `.env.local` - Environment variables
- `config.yaml` - AI service configuration
- `docker-compose.yml` - Service definitions
- `docker-compose-dev.yaml` - Development overrides

### deployment (Production)

**Structure**:
```
deployment/
├── base/                    # Kustomize base
├── overlays/                # Environment-specific overlays
└── charts/                  # Helm charts
```

**Technologies**:
- Kubernetes manifests
- Kustomize for environment management
- Helm for package management

### wren-mdl (JSON Schema)

**Purpose**: Define MDL manifest structure for validation

**Contents**:
- MDL schema JSON (471 LOC)
- Model definitions (tables, columns, relationships)
- Metric definitions
- Calculated field specifications

## Communication Flow

```
User Browser
    ↓ HTTP
Wren UI (Next.js :3000)
    ↓ GraphQL (Apollo Server embedded)
    ├─→ Wren AI Service (:5556) [REST API]
    │       ↓
    │   ├─→ Qdrant (:6333) [Vector search]
    │   └─→ LLM Provider [Text generation]
    │
    ├─→ Wren Engine (:8080) [SQL validation/execution]
    └─→ Ibis Server (:8000) [SQL abstraction]
            ↓
        Data Source (PostgreSQL, BigQuery, etc.)
```

## Data Flow for "Ask" (Text-to-SQL)

1. User submits natural language question in UI
2. UI sends GraphQL mutation to Apollo Server
3. Apollo Server calls AI Service REST API (`/v1/ask`)
4. AI Service runs:
   - Intent classification
   - Context retrieval from Qdrant (RAG)
   - SQL generation via LLM
   - SQL validation against Wren Engine
   - Iterative correction if validation fails
5. Results returned through the chain to UI
6. UI displays SQL, results, and chart

## Development Workflow

### wren-ui
```bash
cd wren-ui
yarn install
yarn dev              # Dev server on :3000 (TZ=UTC)
yarn build            # Production build
yarn lint             # TypeScript + ESLint
yarn test             # Jest unit tests
yarn test:e2e         # Playwright E2E tests
yarn migrate          # Knex migrations
yarn generate-gql     # GraphQL codegen
```

### wren-ai-service
```bash
cd wren-ai-service
poetry install
just init             # Create config.yaml and .env.dev
just up               # Start Docker deps (Qdrant, engine, etc.)
just start            # Run AI service
just test             # pytest
just test-usecases    # Integration tests
just down             # Stop Docker services
```

### wren-launcher
```bash
cd wren-launcher
make build            # Cross-compile
make test             # go test
make check            # fmt + vet + lint
```

## Testing Coverage

| Service | Unit Tests | E2E Tests | Load Tests | Coverage |
|---------|-----------|-----------|------------|----------|
| wren-ui | Jest | Playwright | - | ~60% |
| wren-ai-service | pytest | usecases/ | Locust | ~70% |
| wren-launcher | go test | - | - | ~50% |

## CI/CD

- **Platform**: GitHub Actions
- **Triggers**: PR labels (`ci/ui`, `ci/ai-service`)
- **Docker Registry**: ghcr.io/canner/
- **Tests**: Linting, unit tests, E2E tests on every PR
- **Deployments**: Manual triggers for production

## Key Dependencies

### wren-ui
- `next`: 14.x
- `@apollo/server`: GraphQL server
- `@apollo/client`: GraphQL client
- `knex`: Database ORM
- `antd`: UI components
- `styled-components`: CSS-in-JS

### wren-ai-service
- `fastapi`: Web framework
- `haystack-ai`: RAG framework
- `lite_llm`: LLM gateway
- `qdrant-client`: Vector DB
- `pydantic`: Settings validation
- `hamilton`: Dataflow orchestration

### wren-launcher
- `github.com/docker/compose`: Docker API
- `pterm`: Terminal UI
- `promptui`: Interactive prompts

## File Naming Conventions

- **TypeScript**: kebab-case (`data-source-manager.ts`)
- **Python**: snake_case (`asking_service.py`)
- **Go**: snake_case for files (`root.go`), PascalCase for exports
- **GraphQL**: PascalCase for types (`QueryUser`), camelCase for fields (`userId`)

## Documentation

- **Project Docs**: `./docs/` (this directory)
- **API Docs**: https://wrenai.readme.io/
- **User Guides**: https://docs.getwren.ai/
- **Blog**: https://www.getwren.ai/blog/

## Community

- **Discord**: 1.3k+ members
- **GitHub**: 2.8k+ stars
- **Contributors**: Active open source community

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
