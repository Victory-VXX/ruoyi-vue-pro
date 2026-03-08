# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot 3.5.9 multi-module Maven project (芋道/Ruoyi-Vue-Pro backend) using Java 17. It's an enterprise application framework with modular architecture, supporting features like user management, permissions, workflow, payment, mall, CRM, ERP, and AI integration.

## Build & Run Commands

### Build the project
```bash
# Compile all modules
mvn clean compile

# Package the application (creates yudao-server.jar)
mvn clean package

# Skip tests during build
mvn clean package -DskipTests
```

### Run the application
```bash
# Run the main server application
mvn spring-boot:run -pl yudao-server

# Or run the jar directly after packaging
java -jar yudao-server/target/yudao-server.jar
```

### Testing
```bash
# Run all tests
mvn test

# Run tests for a specific module
mvn test -pl yudao-module-system

# Run a single test class
mvn test -Dtest=UserServiceTest
```

## Architecture

### Module Structure

The project follows a **multi-module Maven architecture** with clear separation of concerns:

**Core Modules:**
- `yudao-dependencies` - Centralized dependency management (BOM)
- `yudao-framework` - Technical components and framework starters
- `yudao-server` - Main application entry point (aggregates business modules)

**Framework Starters** (in `yudao-framework/`):
- `yudao-common` - Common utilities and base classes
- `yudao-spring-boot-starter-mybatis` - MyBatis Plus integration with multi-datasource support
- `yudao-spring-boot-starter-redis` - Redis/Redisson integration
- `yudao-spring-boot-starter-web` - Web layer configuration
- `yudao-spring-boot-starter-security` - Security and authentication
- `yudao-spring-boot-starter-websocket` - WebSocket support
- `yudao-spring-boot-starter-biz-tenant` - Multi-tenancy support
- `yudao-spring-boot-starter-biz-data-permission` - Data permission control
- `yudao-spring-boot-starter-job` - Scheduled job support (Quartz)
- `yudao-spring-boot-starter-mq` - Message queue integration (RocketMQ, Kafka, RabbitMQ)

**Business Modules** (in `yudao-module-*/`):
- `yudao-module-system` - System management (users, roles, permissions, OAuth2, etc.)
- `yudao-module-infra` - Infrastructure services (files, config, code generation, etc.)
- Additional optional modules: `bpm` (workflow), `pay` (payment), `mall` (e-commerce), `crm`, `erp`, `ai`, `iot`, etc.

### Standard Module Package Structure

Each business module follows a consistent layered architecture:

```
yudao-module-{name}/
├── api/                 # API interfaces for other modules to call
│   └── {domain}/        # Domain-specific APIs
├── controller/          # REST API controllers
│   └── {domain}/        # Domain-specific controllers (admin-api paths)
├── convert/            # MapStruct converters (DO ↔ VO/DTO)
├── dal/                # Data Access Layer
│   ├── dataobject/     # Database entity classes (DO)
│   ├── mysql/          # MyBatis Mappers
│   └── redis/          # Redis operations (if needed)
├── service/            # Business logic layer
│   └── {domain}/       # Domain-specific services
├── frameworks/         # Module-specific configurations
├── mq/                 # Message queue consumers/producers
├── job/                # Scheduled jobs
└── enums/              # Enumerations
```

### Key Technologies

- **Framework**: Spring Boot 3.5.9, Spring Security
- **Database**: MyBatis Plus with multi-datasource support (MySQL, PostgreSQL, Oracle, SQL Server, DM, Kingbase, OpenGauss)
- **Cache**: Redis (via Redisson)
- **Scheduler**: Quartz
- **Message Queue**: RocketMQ, Kafka, RabbitMQ
- **API Documentation**: Knife4j (Swagger)
- **Code Generation**: Built-in code generator
- **Multi-tenancy**: Tenant isolation at database level
- **Data Permissions**: Row-level security

### Configuration Profiles

- `application.yaml` - Base configuration
- `application-local.yaml` - Local development (default active profile)
- `application-dev.yaml` - Development environment

Server runs on port **48080** by default.

## Important Conventions

### Database Configuration

Database connections are configured in `application-local.yaml`:
- Primary datasource: MySQL at `192.168.35.131:3306/ruoyi-vue-pro`
- Redis: `192.168.35.131:6379`

To use a different database, modify the `spring.datasource.dynamic.datasource.master` configuration.

### Code Generation

The project has a built-in code generator accessible via:
- Admin UI: Infrastructure → Code Generation
- Configuration: `yudao.codegen.*` properties

Generated code follows the standard module package structure.

### Multi-Tenancy

Multi-tenancy is enabled by default (`yudao.tenant.enable: true`). Tenant context is automatically propagated through:
- HTTP requests (via `tenant_id` header)
- Database queries (automatic filtering)
- Redis cache keys

Ignore tenant isolation for specific tables/URLs via `yudao.tenant.ignore-*` configurations.

### Security & Authentication

- Login endpoints support multiple methods: username/password, SMS, OAuth2 (social login), WeChat mini-program
- JWT tokens for stateless authentication
- API encryption can be enabled (`yudao.api-encrypt.enable: true`)

### API Documentation

Access Swagger UI at: `http://localhost:48080/swagger-ui`

### Testing Strategy

- Unit tests use `yudao-spring-boot-starter-test` (includes test utilities and mock configurations)
- Mapper SQL logging is enabled in debug mode for development
- Test generation can be enabled via `yudao.codegen.unit-test-enable: true`

## Module Dependencies

When adding new features:
1. Check if it belongs in an existing module (system, infra, etc.)
2. For cross-module communication, use the `api/` package pattern
3. Enable modules in `yudao-server/pom.xml` by uncommenting dependencies
4. Most business modules are disabled by default for faster compilation

## Common Patterns

### Service Layer
- Interface in `service/{domain}/`
- Implementation in `service/{domain}/` with `*ServiceImpl` suffix
- Use `@Service` annotation
- Inject mappers from `dal/mysql/{domain}/`

### Controllers
- Located in `controller/{domain}/` or `controller/admin/{domain}/`
- Use `@RestController` and `@RequestMapping("/admin-api/{module}")`
- Return standard responses using `CommonResult<T>`

### Data Objects (DO)
- Located in `dal/dataobject/{domain}/`
- Use MyBatis Plus annotations (`@TableName`, `@TableId`, `@TableField`)
- Logical deletion: deleted=0 (active), deleted=1 (deleted)

### API Interfaces
- For inter-module communication, define interfaces in `api/{domain}/`
- Use Spring Feign or direct injection patterns

## Troubleshooting

### Compilation Issues
- Ensure Java 17 is being used
- Run `mvn clean` before `mvn compile`
- Check that all modules are properly flattened (`mvn flatten:flatten` if needed)

### Startup Issues
- Verify database connection in `application-local.yaml`
- Verify Redis is running and accessible
- Check the main application class: `YudaoServerApplication` in `yudao-server`
- Review logs at: `${user.home}/logs/yudao-server.log`

### Adding New Modules
1. Create module directory following `yudao-module-{name}` pattern
2. Add to root `pom.xml` `<modules>` section
3. Create standard package structure (api, controller, service, dal, etc.)
4. Add dependency to `yudao-server/pom.xml` if needed
