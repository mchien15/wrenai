# WrenAI - Project Roadmap

## Overview

This roadmap outlines the planned development phases and milestones for WrenAI. It reflects the current state of the project and future directions based on community feedback and product vision.

## Current Status

**Release**: v1.0 (Stable)
**Last Updated**: 2026-03-29
**Active Development**: Feature enhancements and optimizations

## Development Phases

### Phase 1: Foundation ✅ (Complete)

**Timeline**: Initial release - Q4 2024
**Status**: ✅ Complete

**Deliverables**:
- ✅ Core text-to-SQL functionality
- ✅ Basic MDL semantic layer
- ✅ 3 data source connectors (PostgreSQL, BigQuery, DuckDB)
- ✅ OpenAI LLM integration
- ✅ Docker Compose local development
- ✅ Basic UI with question asking
- ✅ SQL execution and result display

**Success Metrics**:
- ✅ Working local development setup
- ✅ End-to-end text-to-SQL pipeline
- ✅ 50+ GitHub stars
- ✅ 100+ Discord members

---

### Phase 2: Enhanced RAG & Multi-LLM ✅ (Complete)

**Timeline**: Q4 2024 - Q1 2025
**Status**: ✅ Complete

**Deliverables**:
- ✅ Qdrant vector database integration
- ✅ Advanced RAG pipelines (indexing, retrieval, generation)
- ✅ Multi-LLM support via LiteLLM (9+ providers)
- ✅ MDL schema indexing
- ✅ Historical Q&A indexing
- ✅ SQL pair indexing for few-shot learning
- ✅ Chart generation (Vega/Vega-Lite)
- ✅ 13+ data source connectors

**Success Metrics**:
- ✅ SQL accuracy > 80%
- ✅ Semantic search recall > 75%
- ✅ 5+ LLM provider integrations
- ✅ 500+ GitHub stars
- ✅ 500+ Discord members

---

### Phase 3: Production Readiness 🔄 (In Progress)

**Timeline**: Q1 2025 - Q2 2025
**Status**: 🔄 70% Complete

**Deliverables**:
- ✅ Kubernetes deployment manifests
- ✅ PostgreSQL production database support
- ✅ Comprehensive testing (Jest, Playwright, pytest)
- ✅ API documentation (OpenAPI/Swagger)
- ✅ Error handling and logging
- 🔄 Performance optimization (caching, connection pooling)
- 🔄 Security hardening (authentication, encryption)
- 🔄 Monitoring and observability
- 🔄 Load testing and benchmarking

**Pending**:
- [ ] Authentication system (multi-user support)
- [ ] Role-based access control
- [ ] Rate limiting
- [ ] Query result caching
- [ ] Distributed tracing
- [ ] Backup and restore procedures

**Success Metrics**:
- 🔄 Test coverage > 70%
- 🔄 P95 latency < 10s for queries
- 🔄 100+ concurrent users supported
- Target: 1000+ GitHub stars

---

### Phase 4: Advanced Features 📋 (Planned)

**Timeline**: Q2 2025 - Q3 2025
**Status**: 📋 Not Started

**Deliverables**:
- [ ] Streaming responses for long-running queries
- [ ] Query explanation and breakdown
- [ ] SQL editor with manual refinement
- [ ] Multi-turn conversations (chat interface)
- [ ] Data visualization builder
- [ ] Dashboard creation and sharing
- [ ] Export functionality (CSV, images, PDF)
- [ ] Query history and favorites
- [ ] Collaborative features (shared projects)
- [ ] Webhook integrations

**Success Metrics**:
- User engagement (daily active users)
- Query success rate > 85%
- User retention (30-day)
- 1500+ GitHub stars

---

### Phase 5: Enterprise Features 📋 (Planned)

**Timeline**: Q3 2025 - Q4 2025
**Status**: 📋 Not Started

**Deliverables**:
- [ ] Single sign-on (SSO) integration
- [ ] Audit logging and compliance
- [ ] Data governance policies
- [ ] Multi-tenant isolation
- [ ] Custom LLM fine-tuning
- [ ] Private deployment options
- [ ] SLA and support tiers
- [ ] Advanced analytics and reporting
- [ ] API rate limiting and quotas
- [ ] White-labeling options

**Success Metrics**:
- Enterprise pilot customers
- Revenue targets
- Customer satisfaction (NPS)
- 2000+ GitHub stars

---

### Phase 6: Ecosystem & Integrations 📋 (Planned)

**Timeline**: Q4 2025 - Q1 2026
**Status**: 📋 Not Started

**Deliverables**:
- [ ] REST API for third-party integrations
- [ ] SDK libraries (Python, JavaScript)
- [ ] BI tool integrations (Tableau, Power BI)
- [ ] Data catalog integrations (Alation, Collibra)
- [ ] Workflow integrations (Airflow, dbt)
- [ ] Notification integrations (Slack, Teams)
- [ ] Custom data source SDK
- [ ] Plugin system
- [ ] Community app marketplace

**Success Metrics**:
- API adoption
- Third-party integrations
- Community contributions
- 3000+ GitHub stars

---

## Priority Matrix

### High Priority (Q2 2025)
1. Authentication and multi-user support
2. Performance optimization (caching, indexing)
3. Query explanation feature
4. Comprehensive testing and CI/CD improvements
5. Security hardening

### Medium Priority (Q3 2025)
1. Multi-turn conversations
2. Dashboard and visualization builder
3. Export functionality
4. API and SDK development
5. Data source SDK

### Low Priority (Q4 2025+)
1. Enterprise SSO
2. Advanced analytics
3. Plugin system
4. White-labeling
5. Custom LLM fine-tuning

## Technology Roadmap

### Short-Term (Next 3 months)
- **UI**: Migrate to Next.js 15, React 19
- **AI Service**: Optimize RAG pipelines, add hybrid search
- **Database**: Migration guides, schema improvements
- **Testing**: Increase coverage to 80%+

### Medium-Term (Next 6 months)
- **Performance**: Query result caching, CDN for static assets
- **Scalability**: Horizontal scaling, load balancing
- **Observability**: Distributed tracing, metrics dashboards
- **Security**: OAuth2, encryption at rest

### Long-Term (Next 12 months)
- **Architecture**: Event-driven architecture, message queues
- **AI**: Custom model fine-tuning, multimodal LLMs
- **Platform**: Cloud-native optimizations, edge computing
- **Ecosystem**: Partner integrations, community tools

## Community Goals

### GitHub
- [ ] 1000 stars (Q2 2025)
- [ ] 2000 stars (Q3 2025)
- [ ] 3000 stars (Q4 2025)
- [ ] 50+ external contributors
- [ ] 100+ PRs merged from community

### Discord
- [ ] 2000 members (Q2 2025)
- [ ] 3000 members (Q3 2025)
- [ ] 5000 members (Q4 2025)
- [ ] Active community support program

### Documentation
- [ ] Complete API reference
- [ ] Video tutorials (10+)
- [ ] Interactive examples
- [ ] Multilingual documentation (ES, ZH, JP)

## Risk Mitigation

### Technical Risks
1. **LLM API Rate Limits**: Implement caching, multiple providers, fallback
2. **Vector DB Scaling**: Evaluate Qdrant clustering, consider alternatives
3. **Query Performance**: Optimize RAG pipelines, implement query caching

### Product Risks
1. **SQL Accuracy**: Continuous prompt engineering, user feedback loops
2. **Data Source Complexity**: Prioritize popular sources, provide SDK
3. **User Adoption**: Improve onboarding, add tutorials

### Community Risks
1. **Contributor Burnout**: Clear contribution guidelines, quick PR reviews
2. **Support Load**: Self-service documentation, community moderation
3. **Fork Risk**: Active engagement, clear roadmap, responsive issues

## Success Metrics

### Technical Metrics
- **Query Success Rate**: >85% SQL execution success
- **Response Time**: P95 < 10s for end-to-end queries
- **Uptime**: >99% for production deployments
- **Test Coverage**: >70% for critical paths

### Product Metrics
- **User Engagement**: 30% DAU/MAU ratio
- **Query Volume**: 10K+ queries/day
- **Data Sources**: 15+ production connectors
- **LLM Providers**: 10+ integrations

### Community Metrics
- **GitHub Stars**: 3000+ by end of 2025
- **Discord Members**: 5000+ by end of 2025
- **Contributors**: 50+ active contributors
- **Integrations**: 20+ third-party tools

## Dependencies

### External Dependencies
- **LLM Providers**: OpenAI, Anthropic, Google roadmap alignment
- **Data Sources**: Database/cloud warehouse API stability
- **Open Source**: Framework updates (Next.js, FastAPI, Qdrant)

### Internal Dependencies
- **wren-engine**: SQL engine feature parity
- **wren-mdl**: Schema evolution and backwards compatibility
- **Community Feedback**: Feature prioritization based on demand

## Release Cadence

- **Major Releases**: Quarterly (new features, breaking changes)
- **Minor Releases**: Monthly (enhancements, non-breaking)
- **Patch Releases**: Weekly (bug fixes, security updates)
- **Pre-releases**: As needed for beta testing

## How to Contribute

We welcome community contributions! See our [Contributing Guidelines](../CONTRIBUTING.md) for details.

**Priority Areas for Contribution**:
1. Data source connectors
2. LLM provider integrations
3. Documentation improvements
4. Bug fixes and optimizations
5. Test coverage improvements

## Feedback Loop

- **GitHub Issues**: Feature requests and bug reports
- **GitHub Discussions**: Architecture discussions and RFCs
- **Discord**: Real-time community support
- **Public Roadmap**: [Notion Board](https://wrenai.notion.site/)
- **Monthly Updates**: Blog and Discord announcements

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
**Next Review**: 2026-04-30
