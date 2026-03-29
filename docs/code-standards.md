# WrenAI - Code Standards & Conventions

## General Principles

1. **YAGNI** (You Aren't Gonna Need It) - Implement only what's needed now
2. **KISS** (Keep It Simple, Stupid) - Favor simplicity over complexity
3. **DRY** (Don't Repeat Yourself) - Extract reusable code into modules/functions
4. **File Size Limit**: Keep code files under 200 LOC for optimal context management
5. **Descriptive Naming**: Use long, descriptive names (kebab-case) for files - length is fine, clarity is paramount

## File Naming Conventions

| Language | File Naming | Example | Notes |
|----------|-------------|---------|-------|
| TypeScript | kebab-case | `data-source-manager.ts` | Self-documenting names |
| Python | snake_case | `asking_service.py` | PEP 8 standard |
| Go | snake_case | `root.go` | Go convention |
| GraphQL | PascalCase types, camelCase fields | `QueryUser`, `userId` | GraphQL spec |
| Components | PascalCase | `ModelingPage.tsx` | React convention |

## Service-Specific Standards

### wren-ui (TypeScript/Next.js)

#### File Organization
```
src/
├── apollo/server/
│   ├── resolvers/        # One file per domain (e.g., asking.ts, model.ts)
│   ├── services/         # Business logic layer
│   ├── repositories/     # Data access layer
│   ├── adaptors/         # External service adapters
│   └── types/            # Shared TypeScript types
├── components/
│   └── {feature}/        # Feature-based grouping
└── pages/                # Next.js page routes
```

#### Code Style
- **TypeScript**: Strict mode enabled (`tsconfig.json` strict mode)
- **Imports**: Absolute path aliases (`@/*`, `@server/*`)
- **Components**: Functional components with hooks
- **Styling**: styled-components or Ant Design theming
- **Error Handling**: Try-catch with meaningful error messages
- **Logging**: Structured logging via console/winston

#### Naming Conventions
- **Variables**: camelCase (`const userId = 1`)
- **Constants**: UPPER_SNAKE_CASE (`const MAX_RETRIES = 3`)
- **Functions**: camelCase (`function getUserData()`)
- **Classes**: PascalCase (`class DataSourceManager`)
- **Interfaces**: PascalCase with `I` prefix (`interface IUser`)

#### Best Practices
- Use TypeScript types for all function parameters and return values
- Prefer `const` over `let`, avoid `var`
- Use async/await over Promise chains
- Extract complex logic into services
- Use environment variables for configuration (`.env.local`)
- Implement proper error boundaries in React

#### Testing Standards
- **Unit Tests**: Jest with `*.test.ts` suffix
- **E2E Tests**: Playwright with `*.spec.ts` suffix
- **Coverage**: Aim for >60% coverage
- **Test Structure**: Arrange-Act-Assert pattern

#### GraphQL Conventions
- **Queries**: PascalCase (`QueryGetProjects`)
- **Mutations**: PascalCase with verb prefix (`MutationCreateProject`)
- **Subscriptions**: PascalCase (`SubscriptionProjectUpdated`)
- **Types**: PascalCase (`Project`, `DataSource`)
- **Fields**: camelCase (`projectId`, `dataSourceName`)

### wren-ai-service (Python/FastAPI)

#### File Organization
```
src/
├── pipelines/
│   ├── indexing/         # Indexing pipeline modules
│   ├── retrieval/        # Retrieval pipeline modules
│   └── generation/       # Generation pipeline modules
├── web/v1/
│   ├── services/         # Service layer
│   └── routers/          # FastAPI route handlers
├── core/                 # Base abstractions
└── config.py             # Pydantic settings
```

#### Code Style
- **PEP 8**: Follow Python style guide
- **Type Hints**: Use for all function signatures
- **Docstrings**: Google style docstrings
- **Imports**: Group imports (stdlib, third-party, local)
- **Line Length**: Max 100 characters (soft limit)

#### Naming Conventions
- **Variables**: snake_case (`user_id = 1`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRIES = 3`)
- **Functions**: snake_case (`def get_user_data():`)
- **Classes**: PascalCase (`class DataSourceManager`)
- **Modules**: snake_case (`asking_service.py`)

#### Best Practices
- Use Pydantic models for data validation
- Implement proper exception handling with custom exceptions
- Use dependency injection via FastAPI's `Depends()`
- Log with structured format (JSON preferred)
- Use async/await for I/O operations
- Implement health check endpoints

#### Testing Standards
- **Unit Tests**: pytest with `test_*.py` prefix
- **Integration Tests**: `tests/usecases/` directory
- **Load Tests**: Locust for performance testing
- **Coverage**: Aim for >70% coverage
- **Fixtures**: Use pytest fixtures for common setup

#### Pipeline Conventions
- **Hamilton Dataflows**: Use DAG-based pipeline orchestration
- **Configuration**: Declarative config in `config.yaml`
- **Error Handling**: Retry logic with exponential backoff
- **Logging**: Pipeline step logging for debugging

### wren-launcher (Go)

#### File Organization
```
├── cmd/
│   └── root.go           # CLI entry point
├── pkg/
│   ├── docker/           # Docker-related packages
│   └── dbt/              # dbt conversion packages
└── main.go               # Application entry
```

#### Code Style
- **gofmt**: Run `gofmt` before committing
- **golint**: Follow linting recommendations
- **Effective Go**: Follow Go best practices
- **Package Names**: Short, concise, lowercase

#### Naming Conventions
- **Variables**: camelCase (`userID := 1`)
- **Constants**: PascalCase or camelCase (`const MaxRetries = 3`)
- **Functions**: PascalCase (exported), camelCase (private)
- **Interfaces**: PascalCase with `-er` suffix (`type Runner interface`)
- **Packages**: snake_case, short, descriptive

#### Best Practices
- Use `go mod` for dependency management
- Implement proper error handling (never ignore errors)
- Use `context.Context` for cancellation
- Prefer composition over inheritance
- Use channels for goroutine communication
- Implement graceful shutdown

#### Testing Standards
- **Unit Tests**: `*_test.go` files
- **Table-Driven Tests**: Use for multiple test cases
- **Coverage**: Aim for >50% coverage
- **Benchmarking**: `*_bench.go` for performance tests

## Commit Conventions

### Commit Message Format

Follow **Conventional Commits** specification:

```
type(scope): description

[optional body]

[optional footer]
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `perf`: Performance improvements

### Scopes
- `wren-ui`: Frontend changes
- `wren-ai-service`: AI service changes
- `wren-launcher`: CLI changes
- `docker`: Docker configuration
- `deployment`: Deployment manifests

### Examples
```
feat(wren-ui): add chart export functionality

Implement CSV and image export for generated charts.
Add export buttons to chart component.

Closes #123

fix(wren-ai-service): handle empty MDL validation

Add validation for empty MDL schemas before indexing.
Return meaningful error message to user.

fixes #456

docs: update API documentation

Add new endpoints to API docs and update examples.
```

## Code Review Guidelines

### Before Submitting PR
1. Run linters and fix all issues
2. Run tests and ensure all pass
3. Update documentation if needed
4. Add tests for new functionality
5. Ensure code follows style guide
6. Self-review your changes

### Review Checklist
- [ ] Code follows project conventions
- [ ] No syntax errors or warnings
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No hardcoded values
- [ ] Proper error handling
- [ ] No security vulnerabilities
- [ ] Performance considerations addressed

## Security Standards

### General
- Never commit secrets (API keys, passwords, tokens)
- Use environment variables for sensitive configuration
- Sanitize user inputs to prevent injection attacks
- Use parameterized queries for database access
- Implement proper authentication and authorization
- Keep dependencies up to date

### wren-ui Specific
- Escape user-generated content before rendering
- Validate GraphQL inputs
- Use CSRF protection
- Implement Content Security Policy
- Secure cookies with httpOnly and secure flags

### wren-ai-service Specific
- Validate all API inputs with Pydantic
- Sanitize SQL before execution
- Rate limit API endpoints
- Implement proper authentication
- Use HTTPS in production
- Log security events

## Performance Guidelines

### wren-ui
- Code splitting for large bundles
- Lazy load components where possible
- Optimize images (WebP, compression)
- Use React.memo for expensive components
- Implement virtualization for long lists
- Cache GraphQL queries with Apollo Client

### wren-ai-service
- Use async I/O for database/external calls
- Implement caching for expensive operations
- Use connection pooling for databases
- Profile and optimize hot paths
- Implement rate limiting
- Use pagination for large result sets

## Error Handling Standards

### wren-ui
```typescript
try {
  const result = await serviceCall();
} catch (error) {
  console.error('Operation failed:', error);
  // Show user-friendly error message
  toast.error('Failed to load data. Please try again.');
}
```

### wren-ai-service
```python
try:
    result = await service_call()
except SpecificException as e:
    logger.error(f"Operation failed: {e}")
    raise HTTPException(
        status_code=400,
        detail="User-friendly error message"
    )
```

### wren-launcher
```go
if err != nil {
    log.Printf("Operation failed: %v", err)
    return fmt.Errorf("user-friendly error: %w", err)
}
```

## Logging Standards

### wren-ui
- Use `console.log` for development
- Use structured logging for production
- Include request IDs for tracing
- Log at appropriate levels (error, warn, info, debug)

### wren-ai-service
- Use Python `logging` module
- Structured JSON logs in production
- Include correlation IDs
- Log pipeline steps for debugging

### wren-launcher
- Use `log` package
- Include timestamps and levels
- Log key operations and errors

## Documentation Standards

### Code Comments
- Comment **why**, not **what**
- Use docstrings for functions/classes
- Keep comments up to date
- Avoid obvious comments

### API Documentation
- Use OpenAPI/Swagger for REST APIs
- Use GraphQL schema documentation
- Include request/response examples
- Document error codes

### README Files
- Include setup instructions
- Provide usage examples
- Document configuration options
- Keep up to date

## Testing Standards

### Unit Tests
- Test public APIs
- Mock external dependencies
- Test edge cases
- Use descriptive test names

### Integration Tests
- Test service interactions
- Use test databases
- Clean up after tests
- Test error scenarios

### E2E Tests
- Test critical user flows
- Use realistic test data
- Test across browsers
- Keep tests maintainable

## Dependency Management

### wren-ui (Yarn)
- Use `yarn add` for new dependencies
- Keep dependencies up to date
- Review security advisories
- Remove unused dependencies

### wren-ai-service (Poetry)
- Use `poetry add` for new dependencies
- Lock file committed to repo
- Regular dependency updates
- Review for vulnerabilities

### wren-launcher (Go Modules)
- Use `go get` for new dependencies
- Commit `go.sum` file
- Regular dependency updates
- Review for vulnerabilities

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
