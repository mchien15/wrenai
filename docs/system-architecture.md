# WrenAI - System Architecture

## Overview

WrenAI is a distributed system consisting of three main services communicating via REST and GraphQL APIs. The architecture follows a microservices pattern with clear separation of concerns: UI/presentation, business logic/AI processing, and deployment/CLI operations.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Browser                             │
└─────────────────────────────┬───────────────────────────────────┘
                              │ HTTP
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      Wren UI (Next.js)                           │
│                       Port 3000                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │         Apollo GraphQL Server (Embedded in Next.js)        │  │
│  └───────┬─────────────────────────────────┬─────────────────┘  │
│          │                                 │                     │
└──────────┼─────────────────────────────────┼─────────────────────┘
           │ GraphQL                         │ GraphQL
           ↓                                 ↓
┌──────────────────────┐          ┌──────────────────────────────┐
│  Wren AI Service     │          │    Wren Engine               │
│  (FastAPI)           │          │    (Go)                      │
│  Port 5556           │          │    Port 8080                 │
│                      │          │                              │
│  ┌────────────────┐  │          │  SQL Validation              │
│  │  Ask Pipeline  │  │          │  SQL Execution               │
│  │  - Intent      │  │          │  Data Source Abstraction     │
│  │  - Retrieval   │  │          │                              │
│  │  - Generation  │  │          └──────────────────────────────┘
│  │  - Validation  │  │
│  └───────┬────────┘  │
          │             │
          │ REST        │
          ↓             │
┌───────────────────────┐         ┌──────────────────────────────┐
│      Qdrant           │         │    Ibis Server               │
│   (Vector DB)         │         │    (Python)                  │
│   Port 6333           │         │    Port 8000                 │
│                       │         │                              │
│  - MDL Embeddings     │         │  SQL Abstraction Layer       │
│  - Questions          │         │  Data Source Connectors      │
│  - SQL Pairs          │         │  (13+ sources)               │
└───────────────────────┘         └──────────┬───────────────────┘
                                              │
                                              │ SQL
                                              ↓
                                    ┌──────────────────────┐
                                    │   Data Sources        │
                                    │                      │
                                    │ - PostgreSQL         │
                                    │ - BigQuery           │
                                    │ - Snowflake          │
                                    │ - Redshift           │
                                    │ - etc.               │
                                    └──────────────────────┘
```

## Service Communication Flow

### User Query Flow

```
1. User enters question in UI
   ↓
2. UI sends GraphQL mutation to Apollo Server
   ↓
3. Apollo Server → Wren AI Service (REST: POST /v1/ask)
   ↓
4. AI Service executes Ask Pipeline:
   a. Intent Classification
      - Determine query type (data, chart, explanation)
   b. Context Retrieval (RAG)
      - Semantic search in Qdrant for relevant:
        * MDL schema elements (models, columns, relationships)
        * Similar historical questions
        * SQL pair examples
   c. SQL Generation
      - LLM generates SQL from retrieved context
      - Prompt engineering with MDL constraints
   d. SQL Validation
      - Send to Wren Engine for validation
      - Check syntax, schema compliance
   e. Iterative Correction (if validation fails)
      - Retry generation with error feedback
      - Max retries: `max_sql_correction_retries`
   f. Query Execution (optional)
      - Send validated SQL to data source via Ibis Server
      - Return results
   ↓
5. AI Service returns response to Apollo Server
   ↓
6. Apollo Server returns GraphQL response to UI
   ↓
7. UI displays SQL, results, and/or chart
```

## Wren UI Architecture

### Layer Structure

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (React)                  │
│  ┌───────────────────────────────────────────────┐  │
│  │         Apollo Client (GraphQL Client)         │  │
│  └───────────────────┬───────────────────────────┘  │
└──────────────────────┼───────────────────────────────┘
                       │ GraphQL Query/Mutation
                       ↓
┌─────────────────────────────────────────────────────┐
│              Apollo Server (Next.js API)            │
│  ┌───────────────────────────────────────────────┐  │
│  │              Resolvers Layer                   │  │
│  │  - asking.ts, model.ts, project.ts, etc.      │  │
│  └───────────────────┬───────────────────────────┘  │
│                      ↓                               │
│  ┌───────────────────────────────────────────────┐  │
│  │              Services Layer                    │  │
│  │  - askingService, mdlService, queryService    │  │
│  └───────────────────┬───────────────────────────┘  │
│                      ↓                               │
│  ┌───────────────────────────────────────────────┐  │
│  │           Repositories Layer                   │  │
│  │  - Database access via Knex                    │  │
│  └───────────────────┬───────────────────────────┘  │
│                      ↓                               │
│  ┌───────────────────────────────────────────────┐  │
│  │            Adaptors Layer                      │  │
│  │  - AI Service adaptor (HTTP client)            │  │
│  │  - Engine adaptor (HTTP client)                │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
                       │ HTTP
                       ↓
         ┌─────────────────────────┐
         │   External Services     │
         │   - AI Service          │
         │   - Wren Engine         │
         │   - Ibis Server         │
         └─────────────────────────┘
```

### Key Components

**Resolvers** (`src/apollo/server/resolvers/`)
- Map GraphQL queries/mutations to service calls
- Handle authentication/authorization
- Validate inputs
- Format responses

**Services** (`src/apollo/server/services/`)
- Business logic layer
- Coordinate multiple repository calls
- Implement use cases
- Handle transactions

**Repositories** (`src/apollo/server/repositories/`)
- Data access layer
- SQL queries via Knex
- Database connection management
- Query optimization

**Adaptors** (`src/apollo/server/adaptors/`)
- External service integration
- HTTP client wrappers
- Response transformation
- Error handling

## Wren AI Service Architecture

### 4-Layer Architecture

```
┌─────────────────────────────────────────────────────┐
│              API Layer (FastAPI)                    │
│  ┌───────────────────────────────────────────────┐  │
│  │              Routers                           │  │
│  │  - /v1/ask                                    │  │
│  │  - /v1/ask_details                            │  │
│  │  - /v1/semantics/*                            │  │
│  │  - /v1/generate_chart                         │  │
│  └───────────────────┬───────────────────────────┘  │
└──────────────────────┼───────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────┐
│              Service Layer                          │
│  ┌───────────────────────────────────────────────┐  │
│  │  - AskService                                 │  │
│  │  - SemanticsPreparationService                │  │
│  │  - ChartService                               │  │
│  │  - SqlPairsService                            │  │
│  └───────────────────┬───────────────────────────┘  │
└──────────────────────┼───────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────┐
│              Pipeline Layer (Hamilton)              │
│  ┌───────────────────────────────────────────────┐  │
│  │              Pipelines                         │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │  Indexing Pipelines                      │  │  │
│  │  │  - mdl_schema_indexing                   │  │  │
│  │  │  - question_indexing                     │  │  │
│  │  │  - sql_pair_indexing                     │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │  Retrieval Pipelines                     │  │  │
│  │  │  - semantic_retrieval                    │  │  │
│  │  │  - question_retrieval                    │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │  Generation Pipelines                    │  │  │
│  │  │  - sql_generation                        │  │  │
│  │  │  - chart_generation                      │  │  │
│  │  │  - intent_classification                 │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────┐  │  │
│  │  │  Composite Pipelines                     │  │  │
│  │  │  - ask (main text-to-SQL)                │  │  │
│  │  │  - ask_details (explanation)             │  │  │
│  │  └─────────────────────────────────────────┘  │  │
│  └───────────────────┬───────────────────────────┘  │
└──────────────────────┼───────────────────────────────┘
                       │
                       ↓
┌─────────────────────────────────────────────────────┐
│              Provider Layer                         │
│  ┌───────────────────────────────────────────────┐  │
│  │  - LLM Providers (LiteLLM)                    │  │
│  │    * OpenAI, Azure, Anthropic, Google, etc.  │  │
│  │  - Embedder Providers                         │  │
│  │  - Document Store (Qdrant)                    │  │
│  │  - Engine (Wren Engine HTTP client)           │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Pipeline Architecture Details

#### Indexing Pipelines

**Purpose**: Transform MDL, questions, and SQL pairs into vector embeddings for semantic search

**MDL Schema Indexing**:
```
MDL JSON → Parse → Extract:
  - Models (tables)
  - Columns (fields)
  - Relationships (joins)
  - Metrics
  → Chunk → Embed → Store in Qdrant
```

**Question Indexing**:
```
User Question → Preprocess → Embed → Store in Qdrant
  with metadata: model_id, question, SQL, timestamp
```

**SQL Pair Indexing**:
```
(Question, SQL) pairs → Embed question → Store in Qdrant
  with SQL as metadata for few-shot learning
```

#### Retrieval Pipelines

**Semantic Retrieval**:
```
User Question → Embed → Search Qdrant
  → Return Top-K relevant schema chunks
```

**Question Retrieval**:
```
User Question → Embed → Search Qdrant
  → Return similar historical questions + SQL
```

#### Generation Pipelines

**SQL Generation**:
```
Context (from retrieval) + User Question
  → Prompt Template → LLM → SQL Query
```

**Chart Generation**:
```
Query Result + Data Profile
  → LLM (chart type selection) → Vega Spec
```

**Intent Classification**:
```
User Question → LLM → Intent:
  - data_query (generate SQL)
  - chart_query (generate chart)
  - explanation (explain SQL)
```

#### Ask Pipeline (Main Orchestrator)

```
User Question
  ↓
Intent Classification
  ↓
Context Retrieval (RAG)
  ↓
SQL Generation (with retrieved context)
  ↓
SQL Validation (via Wren Engine)
  ↓
┌─────────────────┐
│ Valid?          │
└────┬──────┬─────┘
     │ Yes   │ No
     ↓       ↓
Execute   Retry Generation (with error feedback)
   │          ↑
   │          └── Max retries reached?
   │                     ↓
   │              Return error
   ↓
Return Results + Chart (optional)
```

## RAG Architecture

### Retrieval-Augmented Generation Flow

```
┌─────────────────────────────────────────────────────┐
│                   Knowledge Base                    │
│  ┌───────────────────────────────────────────────┐  │
│  │  MDL Schema (Business Logic)                  │  │
│  │  - Models, Columns, Relationships             │  │
│  │  - Metrics, Calculated Fields                 │  │
│  │  - Business Definitions                       │  │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────┐  │
│  │  Historical Q&A                              │  │
│  │  - Past user questions                        │  │
│  │  - Generated SQL                              │  │
│  │  - User feedback                              │  │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────┐  │
│  │  SQL Pairs (Few-Shot Examples)               │  │
│  │  - Curated question-SQL pairs                 │  │
│  │  - Common query patterns                      │  │
│  └───────────────────────────────────────────────┘  │
└───────────────────┬─────────────────────────────────┘
                    │ Indexing
                    ↓
         ┌─────────────────────┐
         │   Qdrant (Vector    │
         │   Database)         │
         │                     │
         │  - Embeddings       │
         │  - Metadata         │
         │  - Semantic Search  │
         └──────────┬──────────┘
                    │ Retrieve
                    ↓
┌─────────────────────────────────────────────────────┐
│                   LLM Generation                    │
│  ┌───────────────────────────────────────────────┐  │
│  │  Prompt Engineering                           │  │
│  │  - Retrieved context (schema, examples)       │  │
│  │  - User question                              │  │
│  │  - MDL constraints                            │  │
│  └───────────────────┬───────────────────────────┘  │
│                      ↓                               │
│  ┌───────────────────────────────────────────────┐  │
│  │  LLM Inference                               │  │
│  │  - SQL generation                            │  │
│  │  - Chart generation                          │  │
│  │  - Explanation generation                     │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Why RAG?

**Problem**: LLMs lack domain-specific knowledge about your data schema and business logic.

**Solution**: RAG provides:
1. **Context Awareness**: LLM sees relevant schema elements (tables, columns, relationships)
2. **Few-Shot Learning**: LLM learns from similar historical questions
3. **Business Logic**: MDL encodes metrics, calculated fields, business rules
4. **Accuracy**: Retrieved context grounds LLM generation in actual schema

## Data Models

### MDL (Metadata Definition Language)

**Purpose**: Semantic layer that maps business concepts to database schema.

**Structure**:
```json
{
  "models": [
    {
      "name": "orders",
      "refSql": "SELECT * FROM public.orders",
      "columns": [
        {
          "name": "order_id",
          "type": "integer"
        },
        {
          "name": "total_amount",
          "type": "decimal",
          "description": "Total order amount including tax and shipping"
        }
      ],
      "relationships": [
        {
          "name": "customers",
          "to": "customers",
          "type": "one_to_many",
          "condition": "orders.customer_id = customers.id"
        }
      ],
      "metrics": [
        {
          "name": "total_revenue",
          "expression": "SUM(orders.total_amount)"
        }
      ]
    }
  ]
}
```

**Key Concepts**:
- **Models**: Virtual tables built on SQL queries
- **Columns**: Fields with types and descriptions
- **Relationships**: Joins between models
- **Metrics**: Predefined business calculations
- **Calculated Fields**: Reusable expressions

## Database Architecture

### wren-ui Database

**Type**: SQLite (dev) or PostgreSQL (prod)

**Key Tables**:
- `projects` - User projects
- `data_sources` - Data source connections
- `models` - MDL models
- `questions` - Historical questions
- `dashboards` - User dashboards
- `migrations` - Migration tracking

**ORM**: Knex.js with:
- Connection pooling
- Query builder
- Migration system
- Seed data support

## Deployment Architecture

### Local Development (Docker Compose)

```
┌─────────────────────────────────────────────────────┐
│              Docker Network                         │
│  ┌──────────────────┐  ┌──────────────────────┐    │
│  │  wren-ui         │  │  wren-ai-service     │    │
│  │  :3000           │  │  :5556               │    │
│  └──────────────────┘  └──────────────────────┘    │
│  ┌──────────────────┐  ┌──────────────────────┐    │
│  │  wren-engine     │  │  ibis-server         │    │
│  │  :8080           │  │  :8000               │    │
│  └──────────────────┘  └──────────────────────┘    │
│  ┌──────────────────┐  ┌──────────────────────┐    │
│  │  qdrant          │  │  bootstrap            │    │
│  │  :6333           │  │  (init)               │    │
│  └──────────────────┘  └──────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

### Production (Kubernetes)

```
┌─────────────────────────────────────────────────────┐
│              Kubernetes Cluster                     │
│  ┌───────────────────────────────────────────────┐  │
│  │  Ingress Controller (NGINX/Traefik)           │  │
│  └───────────────────┬───────────────────────────┘  │
│                      ↓                              │
│  ┌───────────────────────────────────────────────┐  │
│  │  Services (LoadBalancer/ClusterIP)            │  │
│  └───────────────────┬───────────────────────────┘  │
│                      ↓                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ wren-ui      │  │ wren-ai      │  │ qdrant   │ │
│  │ Pods (3x)    │  │ service      │  │ Stateful │ │
│  └──────────────┘  │ Pods (3x)    │  │ Set (3x) │ │
│  ┌──────────────┐  └──────────────┘  └──────────┘ │
│  │ wren-engine  │  ┌──────────────┐                │
│  │ Pods (2x)    │  │ ibis-server  │                │
│  └──────────────┘  │ Pods (2x)    │                │
│                    └──────────────┘                │
└─────────────────────────────────────────────────────┘
         │                    │
         ↓                    ↓
    ┌─────────┐         ┌──────────┐
    │ PostgreSQL│        │ Managed  │
    │ (Cloud)  │         │ Vector DB│
    └─────────┘         └──────────┘
```

## Scalability Considerations

### Horizontal Scaling
- **wren-ui**: Stateless, scale via replicas
- **wren-ai-service**: Stateless, scale via replicas
- **wren-engine**: Stateful, limited scaling (connection pooling)
- **qdrant**: StatefulSet with clustering

### Vertical Scaling
- **wren-ai-service**: More CPU for LLM inference
- **qdrant**: More RAM for vector storage
- **wren-ui**: More RAM for GraphQL caching

### Bottlenecks
1. **LLM API Rate Limits**: Mitigate with multiple providers, caching
2. **Vector Search Latency**: Mitigate with Qdrant clustering
3. **Database Connections**: Mitigate with connection pooling
4. **Network I/O**: Mitigate with CDN, edge caching

## Security Architecture

### Authentication/Authorization
- **wren-ui**: Session-based auth, JWT tokens
- **wren-ai-service**: API key authentication
- **Data Sources**: Encrypted credentials at rest

### Network Security
- HTTPS in production
- Internal service communication via mTLS (optional)
- Ingress firewalls and security groups

### Data Security
- Encrypted secrets (vault)
- No sensitive data in logs
- SQL injection prevention (parameterized queries)
- Input validation and sanitization

## Monitoring & Observability

### Health Checks
- `/health` endpoints on all services
- Dependency health checks (database, external APIs)
- Kubernetes liveness/readiness probes

### Logging
- Structured JSON logs
- Correlation IDs for request tracing
- Log aggregation (ELK/Loki)
- Error tracking (Sentry)

### Metrics
- Request latency, error rates
- LLM API call metrics
- Database query performance
- Vector search latency

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
