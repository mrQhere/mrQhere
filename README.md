### Hi, I'm Q 👋
B.Tech ECE (Data Science) @ SRM KTR, class of 2028.
Building toward a security-focused SWE role — AppSec / VAPT.

### 🔧 Main Projects
- **[SecurityManagementPlatform](https://github.com/mrQhere/SecurityManagementPlatform)** —
  local-first, zero-egress VAPT orchestration tool. EPSS/GreyNoise/CISA KEV
  correlation stack — not another tool-count wrapper.
- **[eye-scan](https://github.com/mrQhere/eye-scan)** — offline-first edge AI
  diagnostic PWA. ONNX/WASM on-device inference, Grad-CAM explainability,
  federated learning hooks — zero patient data leaves the device.
- **[stock](https://github.com/mrQhere/stock)** — Indian stock-market analysis
  dashboard (Streamlit/XGBoost/PyTorch).

### 🛠️ Stack
Burp Suite · Nmap · FFUF · Amass · Nuclei · SQLMap · Python · ONNX · Kali/Parrot

### 📌 Currently
Grinding CTFs (picoCTF → HTB/THM) · VAPT dev internship @ CountAI 
---
agent: devin-local
session: occipital-cabbage
created: 2026-08-14T16:29:58Z
---
# SMP Enterprise-Grade Rebuild Plan

Transform the Security Management Platform from a scanner/report application into a properly separated security-data pipeline while preserving working functionality.

## Current Understanding

### Baseline Assessment
- **Version**: V9.4.3 (148 commits)
- **Architecture**: Monolithic scan_runner.py with 55+ scanners, SQLCipher encryption, PySide6 UI, FastAPI API
- **Core Concept**: Nmap discovers services → offline CVE intelligence → verified findings → risk → report
- **Current State**: Working but architecturally problematic - tight coupling between components

### Key Components Identified
1. **main.py** - Entry point with single-instance lock, password protection, UI/API mode selection
2. **scanners/scan_runner.py** - Large monolithic scan orchestrator with DAG execution
3. **scanners/core/registry.py** - Scanner registration system with decorator and auto-discovery
4. **tools/db_manager.py** - Database operations with SQLCipher encryption
5. **tools/encryption_manager.py** - Password-based encryption with PBKDF2
6. **tools/finding_deduplicator.py** - Title-based deduplication (needs replacement)
7. **tools/report_generator.py** - PDF/HTML report generation
8. **intelligence/brain.py** - Global intelligence aggregation
9. **ui/dashboard.py** - PySide6 desktop interface
10. **api/server.py** - FastAPI REST API with JWT auth

### Current Database Structure
- **security.db** - Main encrypted database (engagements, scans, findings, etc.)
- **global_intel.db** - Unencrypted global intelligence (crowdsourced)
- **cve_secondary.db** - CVE data (not encrypted per current design)
- **backup/active_scans.db** - Redundancy database for scan recovery

### Critical Issues to Address
1. Scan runner lacks proper state machine and verification
2. Deduplication is destructive (title-based, loses metadata)
3. No separation between observations and findings
4. CVE intelligence not properly isolated in separate database
5. No evidence abstraction (large outputs in SQLite)
6. Scanner output directly becomes findings (should be observations)
7. No proper scope engine
8. Nmap treated as generic scanner rather than service discovery source

## Open Questions & Clarifications Needed

### Phase 1 Scope & Prioritization
- Should we complete all 10 phases in sequence, or prioritize specific phases?
- What is the timeline expectation for this rebuild?
- Are there specific blockers or critical pain points driving this rebuild?

### Database Migration Strategy
- Should we preserve existing data from security.db during the schema redesign?
- How should we handle the transition from current findings to new observation/finding model?
- What is the strategy for migrating existing scanner outputs to new evidence store?

### Scanner Integration Approach
- Should we incrementally refactor scanners to use new observation model, or create adapters?
- How many of the 55+ scanners are actively used vs. legacy?
- Should we focus on a subset of core scanners first (Nmap, Nuclei, Nikto)?

### UI/UX Transition
- Should the UI be refactored alongside backend changes, or after?
- How do we maintain UI functionality during backend restructuring?
- Should we create new UI screens for the new data model (observations, evidence)?

### Testing Strategy
- What is the current test coverage level?
- Should we add comprehensive tests before refactoring?
- How do we validate that the rebuild preserves existing functionality?

### Intelligence System
- Should we integrate with NVD/CISA KEV/EPSS immediately or phase this?
- What is the priority for offline intelligence capabilities?
- How do we handle the current global_intel.db vs. new vulnerability.db?

### Deployment & Rollout
- Should this be a breaking change (V10.0) or backward compatible?
- How do we handle existing users' data during upgrade?
- What is the rollback strategy if issues arise?

## User Decisions & Constraints

### Confirmed Approach
- **Scope**: Phases 1-3 detailed implementation (Baseline/Security/Storage, Engagement/Scope/Planner, Scanner Execution)
- **Timeline**: Complete all 10 phases sequentially as per master prompt
- **Data Migration**: Fresh start acceptable (data loss acceptable for rebuild)
- **Scanner Migration**: Framework before migration (build observation framework first, then migrate systematically)
- **Evidence Storage**: File storage with metadata (encrypted files with SQLite metadata references only)
- **Backward Compatibility**: Breaking changes acceptable (V10.0 major version bump)
- **UI Timing**: UI refactored after backend is stable

### Current Architecture Summary

#### Database Structure
- **security.db** (encrypted): targets, scans, findings, technologies, risk_scores, raw_scan_output, alerts, logs, baselines
- **cve.db** (unencrypted): CVE data with FTS4 search, CVSS, EPSS, CISA KEV flags
- **global_intel.db** (unencrypted): Crowdsourced global heuristics
- **backup/**: active_scans.db, cve_secondary.db, full_backup.db (mirrors)

#### Scanner Registry
- 86 registered scanners via @register_scanner decorator
- Categories: Passive recon, Active vuln scanning, Network analysis, Web testing, Code analysis, Cloud security
- Core scanners: Nmap, Nuclei, Nikto, SQLMap, Subfinder, HTTPx, WhatWeb, etc.

#### Intelligence Sources
- NVD CVE sync (full + incremental)
- CISA KEV catalog
- EPSS scores (FIRST.org)
- GitHub Security Advisories
- GreyNoise IP intelligence
- MITRE ATT&CK mapping

#### Current Issues
- Monolithic scan_runner.py with process management
- Title-based destructive deduplication
- No observation/finding separation
- CVE data in same database schema as application data
- Large raw outputs stored in SQLite TEXT columns
- Scanner output directly becomes findings
- No proper scope engine
- Minimal test coverage (3 tests in test_security.py)

## Professional Enhancement Additions

### Additional Enterprise-Grade Improvements

Beyond the core 10-phase rebuild, the following professional enhancements will be integrated to ensure SMP V10.0 is production-ready, maintainable, and future-proof:

#### 1. API-First Design Philosophy
- **RESTful API Design**: All functionality accessible via well-designed REST APIs
- **OpenAPI Specification**: Complete OpenAPI 3.0 specification for all endpoints
- **API Versioning**: Proper versioning strategy (/api/v1/, /api/v2/)
- **Rate Limiting**: Per-user and per-endpoint rate limiting
- **API Authentication**: OAuth 2.0 + JWT with token refresh
- **API Documentation**: Interactive API documentation with Swagger UI
- **SDK Generation**: Automatic client SDK generation from OpenAPI spec

#### 2. Plugin & Extension System
- **Plugin Architecture**: Pluggable components for scanners, parsers, exporters
- **Hook System**: Well-defined hooks for extending functionality
- **Plugin Marketplace**: Future capability for plugin distribution
- **Sandboxed Plugins**: Plugin execution in isolated environments
- **Plugin Dependencies**: Automatic dependency resolution for plugins
- **Plugin Validation**: Signature verification and security checks

#### 3. Configuration Management
- **Configuration as Code**: YAML/JSON configuration files
- **Environment-Specific Configs**: dev, staging, production configurations
- **Configuration Validation**: Schema validation for all configurations
- **Secret Management**: Integration with secret stores (HashiCorp Vault, AWS Secrets)
- **Configuration Hot-Reload**: Runtime configuration updates without restart
- **Configuration Migration**: Automated configuration upgrades between versions

#### 4. Monitoring & Observability
- **Metrics Collection**: Prometheus-compatible metrics
- **Distributed Tracing**: OpenTelemetry integration for request tracing
- **Structured Logging**: JSON-structured logs with correlation IDs
- **Health Checks**: Comprehensive health check endpoints
- **Performance Monitoring**: Database query performance, API response times
- **Alerting Integration**: Integration with alerting systems (PagerDuty, Slack)
- **Dashboard Integration**: Grafana dashboards for system monitoring

#### 5. Comprehensive Testing Infrastructure
- **Unit Testing**: pytest-based unit tests with >80% coverage
- **Integration Testing**: Database and external service integration tests
- **End-to-End Testing**: Full pipeline testing with test fixtures
- **Property-Based Testing**: Hypothesis-based property testing
- **Performance Testing**: Load testing and performance profiling
- **Security Testing**: Automated security scanning (SAST, DAST)
- **Contract Testing**: API contract testing with Pact
- **Test Data Management**: Test data factories and fixtures

#### 6. Documentation Excellence
- **Auto-Generated API Docs**: API documentation from OpenAPI spec
- **Architecture Diagrams**: C4 model architecture diagrams
- **Decision Records**: Architecture Decision Records (ADRs)
- **Runbooks**: Operational runbooks for common procedures
- **Troubleshooting Guides**: Detailed troubleshooting procedures
- **Code Documentation**: Comprehensive docstrings with examples
- **User Documentation**: User guides with screenshots and tutorials

#### 7. Performance Optimization
- **Database Query Optimization**: Query optimization, proper indexing
- **Caching Layer**: Redis caching for frequently accessed data
- **Async Operations**: Async/await for I/O-bound operations
- **Connection Pooling**: Database connection pooling
- **Lazy Loading**: On-demand data loading for large datasets
- **Batch Processing**: Efficient batch operations for bulk data
- **Memory Management**: Proper memory management and cleanup

#### 8. Security Hardening
- **Input Validation**: Comprehensive input validation and sanitization
- **Output Encoding**: Proper output encoding to prevent XSS
- **CSRF Protection**: Cross-Site Request Forgery protection
- **Security Headers**: Security headers (CSP, HSTS, X-Frame-Options)
- **Dependency Scanning**: Automated dependency vulnerability scanning
- **Secret Scanning**: Automatic secret detection in code
- **Penetration Testing**: Regular penetration testing
- **Security Audits**: Regular security audits and reviews

#### 9. Data Validation & Integrity
- **Schema Validation**: JSON Schema validation for all data
- **Data Integrity Checks**: Regular data integrity verification
- **Referential Integrity**: Database referential integrity enforcement
- **Data Validation Rules**: Business rule validation
- **Data Sanitization**: Automatic data sanitization and normalization
- **Data Migration**: Automated data migration between schema versions
- **Data Backup Verification**: Regular backup verification and testing

#### 10. Error Handling & Resilience
- **Graceful Degradation**: System continues with reduced functionality on errors
- **Circuit Breakers**: Circuit breaker pattern for external service calls
- **Retry Logic**: Exponential backoff retry for transient failures
- **Error Taxonomy**: Comprehensive error classification and handling
- **Error Recovery**: Automatic error recovery where possible
- **Error Reporting**: Detailed error reporting with context
- **User-Friendly Errors**: Clear, actionable error messages for users

#### 11. Internationalization & Accessibility
- **i18n Support**: Multi-language support with gettext
- **Unicode Support**: Full Unicode support throughout
- **Timezone Handling**: Proper timezone handling and conversion
- **Date/Time Formatting**: Locale-aware date/time formatting
- **Accessibility**: WCAG 2.1 AA compliance for UI
- **Keyboard Navigation**: Full keyboard navigation support
- **Screen Reader Support**: Screen reader compatibility
- **High Contrast Mode**: High contrast mode support

#### 12. Deployment & Operations
- **Containerization**: Docker containerization with multi-stage builds
- **Orchestration**: Kubernetes deployment manifests
- **CI/CD Pipeline**: GitHub Actions/Jenkins CI/CD pipeline
- **Infrastructure as Code**: Terraform/Ansible for infrastructure
- **Blue-Green Deployment**: Zero-downtime deployment strategy
- **Rollback Capability**: One-click rollback capability
- **Health Monitoring**: Container health monitoring
- **Log Aggregation**: Centralized log aggregation (ELK stack)

#### 13. Backup & Disaster Recovery
- **Automated Backups**: Scheduled automated backups
- **Incremental Backups**: Incremental backup support
- **Backup Encryption**: Backup encryption at rest and in transit
- **Backup Verification**: Regular backup verification and testing
- **Disaster Recovery Plan**: Comprehensive disaster recovery procedures
- **Recovery Time Objectives**: Defined RTO and RPO
- **Geo-Redundancy**: Geographic redundancy for critical data
- **Recovery Testing**: Regular disaster recovery testing

#### 14. Scalability Architecture
- **Horizontal Scaling**: Horizontal scaling capability
- **Load Balancing**: Load balancing for API servers
- **Database Sharding**: Database sharding capability for large datasets
- **Message Queue**: Message queue for async processing (RabbitMQ/Kafka)
- **Task Queue**: Background task processing (Celery)
- **Caching Layer**: Distributed caching (Redis Cluster)
- **CDN Integration**: CDN integration for static assets
- **Auto-scaling**: Auto-scaling based on load

#### 15. AI/LLM Integration Framework
- **AI Plugin Architecture**: Pluggable AI/LLM providers
- **Prompt Templates**: Managed prompt templates for consistency
- **AI Result Validation**: Validation of AI-generated content
- **Fallback Mechanisms**: Fallback to deterministic logic on AI failure
- **Privacy-Preserving AI**: Local AI options, data minimization
- **AI Usage Tracking**: AI usage monitoring and cost tracking
- **AI Rate Limiting**: Rate limiting for AI API calls
- **Deterministic Priority**: AI as enhancement, never authoritative

#### 16. Workflow Automation
- **Workflow Engine**: Workflow engine for custom automation
- **Trigger System**: Event-based trigger system
- **Action System**: Pluggable action system
- **Condition Builder**: Visual condition builder for workflows
- **Workflow Templates**: Pre-built workflow templates
- **Workflow Scheduling**: Scheduled workflow execution
- **Workflow Monitoring**: Workflow execution monitoring
- **Workflow Versioning**: Workflow versioning and rollback

#### 17. Compliance Framework
- **Compliance Mappings**: Built-in compliance mappings (SOC 2, ISO 27001, PCI-DSS, HIPAA, GDPR)
- **Compliance Reporting**: Automated compliance report generation
- **Audit Trail Enhancement**: Immutable audit logs with blockchain hashing
- **Data Retention**: Configurable data retention policies
- **Data Classification**: Data classification and handling
- **Privacy Controls**: GDPR-compliant privacy controls
- **Consent Management**: User consent management
- **Right to be Forgotten**: Data deletion capabilities

#### 18. Developer Experience
- **CLI Tool**: Comprehensive CLI tool for all operations
- **SDK Libraries**: Client SDK libraries (Python, JavaScript, Go)
- **Webhooks**: Webhook support for event notifications
- **Websocket API**: Real-time updates via WebSockets
- **Developer Portal**: Developer portal with documentation and tools
- **Sandbox Environment**: Developer sandbox environment
- **API Explorer**: Interactive API exploration tool
- **Sample Code**: Comprehensive sample code and tutorials

#### 19. Data Export & Integration
- **Export Formats**: Multiple export formats (JSON, CSV, XML, PDF, HTML)
- **API Integration**: Easy integration with other tools via API
- **Webhook Integration**: Webhook notifications for events
- **Data Synchronization**: Data synchronization with external systems
- **Import/Export**: Bulk import/export capabilities
- **Data Transformation**: Data transformation and mapping
- **Scheduled Exports**: Scheduled automated exports
- **Custom Export Templates**: Custom export template support

#### 20. Performance Profiling & Optimization
- **Built-in Profiler**: Built-in performance profiling tools
- **Query Analysis**: Database query analysis and optimization
- **Memory Profiling**: Memory profiling and leak detection
- **Performance Baselines**: Performance baseline tracking
- **Performance Regression Testing**: Automated performance regression testing
- **Slow Query Detection**: Automatic slow query detection
- **Performance Dashboards**: Performance monitoring dashboards
- **Optimization Recommendations**: Automatic optimization recommendations

### AI/LLM Integration Considerations

#### AI System Design Principles

**1. Clear Data Structures for AI Understanding**
- Well-defined schemas with type hints
- Consistent naming conventions
- Comprehensive documentation
- Example data for each schema
- Clear relationships between entities

**2. Non-Authoritative AI Integration**
- AI suggestions always labeled as such
- Deterministic logic as primary decision maker
- AI results require validation
- Fallback to deterministic logic on AI failure
- AI confidence scores displayed to users
- User control over AI influence level

**3. Privacy-Preserving AI Interactions**
- Local AI model support (Ollama, LocalLLM)
- Data minimization for AI API calls
- PII redaction before AI processing
- Option to disable AI features
- AI usage transparency
- Data retention policies for AI data

**4. Exact Mapping and Precision**
- Schema versioning for AI compatibility
- API contracts with exact specifications
- Test fixtures for AI validation
- Deterministic input/output examples
- Clear error handling for AI failures
- Comprehensive logging of AI decisions

**5. Weak Model Compatibility**
- Simple, clear prompts for weak models
- Fallback to rule-based logic
- Progressive enhancement approach
- Model capability detection
- Graceful degradation on model limitations
- Clear documentation of model requirements

#### AI Integration Points

**1. Finding Description Enhancement**
- AI assists in writing clear descriptions
- Technical to business language translation
- Consistent terminology enforcement
- Template-based description generation
- Human review and editing capability

**2. Remediation Suggestions**
- AI suggests remediation steps
- Priority-based remediation recommendations
- Context-aware remediation advice
- Integration with vulnerability databases
- Human verification required

**3. Risk Assessment Assistance**
- AI assists in risk factor analysis
- Environmental context consideration
- Asset criticality integration
- Trend analysis and prediction
- Human oversight for final decisions

**4. Report Generation Enhancement**
- AI assists in executive summary writing
- Technical to executive translation
- Consistency in report language
- Template-based report sections
- Human review and editing

**5. Query Assistance**
- Natural language query interface
- AI-assisted complex query building
- Query optimization suggestions
- Result explanation and interpretation
- Exact query logging for reproducibility

#### AI Implementation Architecture

```python
class AIIntegrationLayer:
    """
    AI integration layer with fallback mechanisms and validation.
    """
    
    def __init__(self, config: dict):
        self.config = config
        self.ai_provider = self._initialize_provider()
        self.fallback_logic = FallbackLogic()
    
    def enhance_finding_description(self, finding: dict) -> str:
        """
        Enhance finding description with AI assistance.
        Falls back to template-based generation if AI fails.
        """
        try:
            ai_suggestion = self.ai_provider.generate_description(finding)
            validated = self._validate_ai_output(ai_suggestion, finding)
            if validated:
                return self._add_ai_attribution(ai_suggestion)
            else:
                return self.fallback_logic.generate_description(finding)
        except Exception as e:
            logger.warning(f"AI enhancement failed: {e}, using fallback")
            return self.fallback_logic.generate_description(finding)
    
    def suggest_remediation(self, finding: dict) -> list[str]:
        """
        Suggest remediation steps with AI assistance.
        Always requires human verification.
        """
        try:
            ai_suggestions = self.ai_provider.suggest_remediation(finding)
            validated = self._validate_remediation(ai_suggestions, finding)
            return self._add_verification_required(validated)
        except Exception as e:
            logger.warning(f"AI remediation failed: {e}, using database")
            return self.fallback_logic.get_remediation_from_db(finding)
    
    def _validate_ai_output(self, output: str, context: dict) -> bool:
        """Validate AI output against context and constraints."""
        pass
    
    def _add_ai_attribution(self, content: str) -> str:
        """Add AI attribution to generated content."""
        return f"[AI-Assisted] {content}"
    
    def _add_verification_required(self, suggestions: list) -> list:
        """Mark suggestions as requiring human verification."""
        return [f"[HUMAN VERIFICATION REQUIRED] {s}" for s in suggestions]
```

### Implementation Integration

These enhancements will be integrated throughout the 10 phases:

- **Phase 1**: Add API-first design, configuration management, monitoring foundations
- **Phase 2**: Add workflow automation, compliance framework foundations
- **Phase 3**: Add performance profiling, error handling enhancements
- **Phase 4**: Add AI integration for intelligence analysis
- **Phase 5**: Add data validation, integrity checks
- **Phase 6**: Add scalability architecture, async operations
- **Phase 7**: Add AI-assisted deduplication, workflow integration
- **Phase 8**: Add AI-enhanced service discovery, performance optimization
- **Phase 9**: Add accessibility, internationalization, developer experience
- **Phase 10**: Add AI-assisted reporting, export enhancements, compliance reporting

Each enhancement will be implemented with the same rigor as the core phases, including testing, documentation, and acceptance criteria.

## Complete Architecture Overview

### SMP V10.0 Enterprise Architecture

The Security Management Platform V10.0 represents a complete architectural transformation from a monolithic scanner application to a properly separated security-data pipeline. The new architecture follows strict separation of concerns, immutable data principles, and enterprise-grade security practices.

### Core Architectural Principles

1. **Data Pipeline Philosophy**: Scanner output → Observations → Evidence → Findings → Risk → Reports
2. **Immutability**: Observations and evidence are never modified, only referenced
3. **Separation of Concerns**: Each subsystem has distinct responsibilities and boundaries
4. **Security by Design**: Encryption at rest, proper key management, audit trails
5. **Offline-First**: Full capability without internet connectivity after initial intelligence download
6. **Scope Authority**: All scanning decisions go through centralized scope engine
7. **Evidence Preservation**: All raw scanner outputs preserved with provenance
8. **Deterministic Behavior**: Repeatable results without "AI magic" for security decisions

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         SMP V10.0 Architecture                   │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   UI Layer       │    │   API Layer      │    │  CLI Layer       │
│  (PySide6)       │    │   (FastAPI)      │    │  (Future)        │
└────────┬─────────┘    └────────┬─────────┘    └────────┬─────────┘
         │                        │                        │
         └────────────────────────┼────────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │   Application Layer      │
                    │  (Engagement, Scope,      │
                    │   Scan Management)       │
                    └─────────────┬─────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
┌───────▼────────┐    ┌──────────▼──────────┐    ┌────────▼────────┐
│  Scope Engine  │    │   Scan Planner      │    │  Scheduler     │
│  (Authorization│    │   (Dependency       │    │  (Execution    │
│   Boundaries)  │    │    Resolution)      │    │   Orchestration)│
└───────┬────────┘    └──────────┬──────────┘    └────────┬────────┘
        │                        │                        │
        └────────────────────────┼────────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │  Execution Sandbox     │
                    │  (Process Management,   │
                    │   Resource Control)    │
                    └────────────┬────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
┌───────▼────────┐    ┌─────────▼─────────┐    ┌────────▼────────┐
│ Scanner        │    │   Observation     │    │  Evidence       │
│ Adapters       │    │   Parsers         │    │  Store          │
│ (86+ Scanners) │    │   (Normalization) │    │  (Encrypted     │
└───────┬────────┘    └─────────┬─────────┘    │   Files)         │
        │                      │              └────────┬────────┘
        │                      │                       │
        └──────────────────────┼───────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Finding Engine   │
                    │  (Correlation,     │
                    │   Deduplication,   │
                    │   Risk Scoring)    │
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
┌───────▼────────┐    ┌─────────▼─────────┐    ┌────────▼────────┐
│ Vulnerability  │    │   Report          │    │  Risk Engine    │
│ Intelligence  │    │   Generator       │    │  (Scoring,      │
│ (Offline CVE   │    │   (PDF/HTML/JSON) │    │   Trending)     │
│  Database)     │    └─────────┬─────────┘    └────────┬────────┘
└───────┬────────┘              │                      │
        │                      │                      │
        └──────────────────────┼──────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Storage Layer     │
                    │  (security.db,      │
                    │   vulnerability.db, │
                    │   evidence files)   │
                    └─────────────────────┘
```

### Data Flow Architecture

```
1. Target Definition
   ↓
2. Scope Validation (Scope Engine)
   ↓
3. Scan Planning (Planner + DAG)
   ↓
4. Scanner Execution (Adapters + Sandbox)
   ↓
5. Raw Output Capture (Evidence Store)
   ↓
6. Observation Parsing (Parsers)
   ↓
7. Observation Storage (Immutable)
   ↓
8. Finding Correlation (Finding Engine)
   ↓
9. CVE Intelligence Matching (Vulnerability DB)
   ↓
10. Risk Scoring (Risk Engine)
    ↓
11. Report Generation (Exporter)
```

### Subsystem Responsibilities

#### 1. Scope Engine
- **Responsibility**: Authorize all scanning operations
- **Input**: Target, scanner activity level
- **Output**: Allow/deny decision with reason
- **Key Features**: CIDR matching, wildcard domains, regex patterns, redirect tracking

#### 2. Scan Planner
- **Responsibility**: Create optimal scan execution plans
- **Input**: Target, policy, scope
- **Output**: Dependency-resolved execution graph
- **Key Features**: Phase-based planning, resource estimation, timeline prediction

#### 3. Scheduler
- **Responsibility**: Orchestrate scanner execution
- **Input**: Execution plan, resource constraints
- **Output**: Coordinated scanner runs
- **Key Features**: DAG execution, concurrency control, pause/resume

#### 4. Execution Sandbox
- **Responsibility**: Isolate and control scanner processes
- **Input**: Scanner adapter, target, configuration
- **Output**: Captured process output
- **Key Features**: Process tracking, resource limits, timeout handling

#### 5. Scanner Adapters
- **Responsibility**: Interface with external security tools
- **Input**: Target, configuration
- **Output**: Raw scanner output
- **Key Features**: Binary verification, execution management, error handling

#### 6. Observation Parsers
- **Responsibility**: Convert raw output to structured observations
- **Input**: Raw scanner output, scanner context
- **Output**: Normalized observations
- **Key Features**: Schema validation, data normalization, confidence scoring

#### 7. Evidence Store
- **Responsibility**: Preserve all raw scanner outputs
- **Input**: Raw output, metadata
- **Output**: Encrypted evidence files
- **Key Features**: Immutable storage, encryption, provenance tracking

#### 8. Finding Engine
- **Responsibility**: Correlate observations into security findings
- **Input**: Observations, CVE intelligence
- **Output**: Correlated findings
- **Key Features**: Deduplication, CVE matching, confidence calculation

#### 9. Vulnerability Intelligence
- **Responsibility**: Maintain offline CVE/CPE/KEV/EPSS database
- **Input**: External intelligence sources
- **Output**: Local vulnerability database
- **Key Features**: NVD sync, CISA KEV, EPSS scoring, offline matching

#### 10. Risk Engine
- **Responsibility**: Calculate and track risk scores
- **Input**: Findings, asset criticality, environmental factors
- **Output**: Risk scores and trends
- **Key Features**: Multi-factor scoring, trend analysis, risk aggregation

#### 11. Report Generator
- **Responsibility**: Generate professional security reports
- **Input**: Findings, evidence, risk scores
- **Output**: PDF/HTML/JSON reports
- **Key Features**: Executive summaries, technical details, compliance mapping

### Database Architecture

#### security.db (Application State)
- **Purpose**: Store all engagement, scan, and finding data
- **Encryption**: AES-256 via SQLCipher with separate DEK
- **Key Tables**: users, engagements, scope_rules, targets, scans, assets, services, observations, findings, audit_log

#### vulnerability.db (Intelligence)
- **Purpose**: Store CVE/CPE/KEV/EPSS intelligence
- **Encryption**: AES-256 via SQLCipher with separate IEK
- **Key Tables**: vulnerabilities, cpe, cpe_match, cvss, epss, kev, references
- **Offline Capability**: Full functionality without internet after initial sync

#### Evidence Storage (File System)
- **Purpose**: Store large raw outputs and artifacts
- **Encryption**: Per-file encryption with EEK
- **Structure**: evidence/<engagement>/<scan>/<evidence_id>/
- **Types**: raw_output, http_requests, http_responses, screenshots, certificates

### Security Architecture

#### Key Hierarchy
```
Master Password (user-provided)
    ↓ PBKDF2-SHA256 (600,000 iterations)
Key Encryption Key (KEK)
    ↓
├── Database Encryption Key (DEK) → security.db
├── Intelligence Encryption Key (IEK) → vulnerability.db
└── Evidence Encryption Key (EEK) → evidence files
```

#### Security Features
- **Encryption at Rest**: All sensitive data encrypted
- **Key Rotation**: Support for periodic key rotation
- **Audit Trail**: All security-sensitive transitions logged
- **Scope Enforcement**: Central authorization for all scanning
- **Provenance Tracking**: Complete execution history
- **Offline Capability**: No external dependencies for core operations

### Migration Strategy

#### V9.4.3 → V10.0 Transition
- **Breaking Changes**: Major version bump, data loss acceptable
- **Database Migration**: Fresh schema, no data migration
- **Scanner Migration**: Framework first, then incremental scanner adaptation
- **UI Migration**: Backend complete before UI refactoring
- **Rollback**: Export functionality for data preservation

## Detailed Implementation Plan: All 10 Phases

### PHASE 1 — BASELINE, SECURITY, STORAGE & CONTRACTS

**Objective**: Create stable foundation with proper data contracts, separated databases, and security model.

#### 1.1 Database Schema Redesign

**Current State Analysis**:
- Single security.db contains everything (engagements, findings, CVEs mixed)
- No separation between operational data and intelligence data
- Raw outputs stored in TEXT columns (bloat, performance issues)
- Missing observation concept

**New Database Architecture**:

```
data/
├── security.db              # Encrypted: Application state
│   ├── users               # Single admin user for now
│   ├── engagements         # Project/engagement metadata
│   ├── scope_rules         # Authorization boundaries
│   ├── targets             # Scan targets
│   ├── scans               # Scan execution records
│   ├── assets              # Discovered assets (from Nmap)
│   ├── services            # Discovered services (from Nmap)
│   ├── observations        # Raw scanner outputs (immutable)
│   ├── findings            # Correlated security findings
│   ├── finding_evidence    # Links findings to observations
│   ├── scan_scanner_status # Per-scanner execution states
│   └── audit_log           # Security-sensitive transitions
│
├── vulnerability.db        # Encrypted: CVE/CPE/KEV/EPSS intelligence
│   ├── vulnerabilities     # Core CVE data
│   ├── aliases             # CVE aliases
│   ├── cwe                 # CWE mappings
│   ├── cvss                # CVSS scores (multiple versions)
│   ├── cpe                 # CPE dictionary
│   ├── cpe_match           # CVE-CPE relationships
│   ├── fixed_version       # Remediation version info
│   ├── epss                # Exploit Prediction Scoring
│   ├── kev                 # CISA Known Exploited Vulnerabilities
│   ├── references          # External references
│   └── intel_import        # Import metadata and rollback
│
├── evidence/               # Encrypted file storage
│   └── <engagement>/<scan>/<evidence_id>
│       ├── raw_output/
│       ├── http_requests/
│       ├── http_responses/
│       ├── screenshots/
│       └── certificates/
│
└── work/                   # Temporary scanner workspace
    └── <scan_id>/
```

**Schema Definitions**:

**security.db - Core Tables**:

```sql
-- Users table (single admin for now, extensible for multi-user)
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,  -- Argon2id
    salt TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'admin',
    created_at TEXT NOT NULL,
    last_login TEXT,
    enabled INTEGER DEFAULT 1
);

-- Engagements table (project/grouping)
CREATE TABLE engagements (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    start_date TEXT NOT NULL,
    end_date TEXT,
    status TEXT NOT NULL DEFAULT 'active',  -- active, completed, archived
    created_by INTEGER NOT NULL,  -- user_id
    created_at TEXT NOT NULL,
    FOREIGN KEY (created_by) REFERENCES users(id)
);

-- Scope rules (authorization boundaries)
CREATE TABLE scope_rules (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    engagement_id INTEGER NOT NULL,
    rule_type TEXT NOT NULL,  -- domain, subdomain, ip, cidr, url, port
    rule_value TEXT NOT NULL,
    action TEXT NOT NULL DEFAULT 'allow',  -- allow, deny
    priority INTEGER DEFAULT 100,
    created_at TEXT NOT NULL,
    FOREIGN KEY (engagement_id) REFERENCES engagements(id)
);

-- Targets table
CREATE TABLE targets (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    engagement_id INTEGER NOT NULL,
    target TEXT NOT NULL,  -- URL, IP, domain
    target_type TEXT NOT NULL,  -- url, ip, domain, cidr
    status TEXT NOT NULL DEFAULT 'enabled',
    added_date TEXT NOT NULL,
    last_scan TEXT,
    notes TEXT,
    FOREIGN KEY (engagement_id) REFERENCES engagements(id)
);

-- Scans table
CREATE TABLE scans (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    engagement_id INTEGER NOT NULL,
    target_id INTEGER NOT NULL,
    scan_type TEXT NOT NULL,  -- standard, osint, full, custom
    profile TEXT NOT NULL,     -- scanner profile configuration
    start_time TEXT NOT NULL,
    end_time TEXT,
    status TEXT NOT NULL,  -- using new state machine
    scanned_by INTEGER NOT NULL,  -- user_id
    report_hash TEXT,
    scanner_count INTEGER DEFAULT 0,
    completed_scanners INTEGER DEFAULT 0,
    failed_scanners INTEGER DEFAULT 0,
    FOREIGN KEY (engagement_id) REFERENCES engagements(id),
    FOREIGN KEY (target_id) REFERENCES targets(id),
    FOREIGN KEY (scanned_by) REFERENCES users(id)
);

-- Assets table (from Nmap service discovery)
CREATE TABLE assets (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    scan_id INTEGER NOT NULL,
    engagement_id INTEGER NOT NULL,
    asset_type TEXT NOT NULL,  -- host, ip, domain, subdomain
    asset_value TEXT NOT NULL,
    confidence REAL DEFAULT 1.0,
    discovered_at TEXT NOT NULL,
    source_scanner TEXT NOT NULL,
    FOREIGN KEY (scan_id) REFERENCES scans(id),
    FOREIGN KEY (engagement_id) REFERENCES engagements(id)
);

-- Services table (from Nmap)
CREATE TABLE services (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    asset_id INTEGER NOT NULL,
    scan_id INTEGER NOT NULL,
    port INTEGER NOT NULL,
    protocol TEXT NOT NULL,  -- tcp, udp
    state TEXT NOT NULL,    -- open, closed, filtered
    service_name TEXT,
    product TEXT,
    version TEXT,
    banner TEXT,
    confidence REAL DEFAULT 0.95,
    discovered_at TEXT NOT NULL,
    FOREIGN KEY (asset_id) REFERENCES assets(id),
    FOREIGN KEY (scan_id) REFERENCES scans(id)
);

-- Observations table (THE CORE DATA CONTRACT)
CREATE TABLE observations (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    observation_id TEXT UNIQUE NOT NULL,  -- UUID
    scan_id INTEGER NOT NULL,
    asset_id INTEGER,
    service_id INTEGER,
    scanner_id TEXT NOT NULL,
    scanner_version TEXT,
    observation_type TEXT NOT NULL,  -- asset, port, service, technology, vulnerability_candidate, etc.
    title TEXT NOT NULL,
    raw_value TEXT,                  -- JSON string of raw data
    normalized_value TEXT,           -- JSON string of normalized data
    confidence REAL DEFAULT 0.5,
    observed_at TEXT NOT NULL,
    raw_output_hash TEXT,
    parser_version TEXT,
    FOREIGN KEY (scan_id) REFERENCES scans(id),
    FOREIGN KEY (asset_id) REFERENCES assets(id),
    FOREIGN KEY (service_id) REFERENCES services(id)
);

-- Findings table (correlated security statements)
CREATE TABLE findings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    finding_id TEXT UNIQUE NOT NULL,  -- UUID
    engagement_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    vulnerability_class TEXT,
    cwe_id TEXT,
    cve_id TEXT,
    asset_id INTEGER,
    service_id INTEGER,
    endpoint TEXT,
    parameter TEXT,
    severity TEXT NOT NULL,
    confidence REAL DEFAULT 0.5,
    status TEXT NOT NULL DEFAULT 'open',  -- open, in_progress, resolved, risk_accepted, false_positive
    risk_score REAL,
    remediation TEXT,
    validation TEXT,
    first_observed_at TEXT NOT NULL,
    last_observed_at TEXT NOT NULL,
    provenance TEXT,  -- JSON string of source information
    FOREIGN KEY (engagement_id) REFERENCES engagements(id),
    FOREIGN KEY (asset_id) REFERENCES assets(id),
    FOREIGN KEY (service_id) REFERENCES services(id)
);

-- Finding evidence mapping
CREATE TABLE finding_evidence (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    finding_id TEXT NOT NULL,
    observation_id TEXT NOT NULL,
    correlation_strength REAL DEFAULT 1.0,
    mapped_at TEXT NOT NULL,
    FOREIGN KEY (finding_id) REFERENCES findings(finding_id),
    FOREIGN KEY (observation_id) REFERENCES observations(observation_id)
);

-- Scanner execution status (new state machine)
CREATE TABLE scan_scanner_status (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    scan_id INTEGER NOT NULL,
    scanner_name TEXT NOT NULL,
    status TEXT NOT NULL,  -- NOT_STARTED, BLOCKED, DEPENDENCY_MISSING, STARTED, RUNNING, COMPLETED, COMPLETED_WITH_FINDINGS, COMPLETED_NO_FINDINGS, FAILED, TIMEOUT, CANCELLED, PARSE_FAILED, PARTIAL, SKIPPED
    start_time TEXT,
    end_time TEXT,
    exit_code INTEGER,
    timeout_reason TEXT,
    retry_count INTEGER DEFAULT 0,
    binary_version TEXT,
    command_hash TEXT,
    raw_output_path TEXT,
    error_message TEXT,
    FOREIGN KEY (scan_id) REFERENCES scans(id)
);

-- Audit log for security-sensitive transitions
CREATE TABLE audit_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    action TEXT NOT NULL,
    resource_type TEXT NOT NULL,
    resource_id TEXT,
    old_value TEXT,
    new_value TEXT,
    ip_address TEXT,
    user_agent TEXT,
    timestamp TEXT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**vulnerability.db - Intelligence Tables**:

```sql
-- Core vulnerability data
CREATE TABLE vulnerabilities (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    internal_id TEXT UNIQUE NOT NULL,
    cve_id TEXT UNIQUE NOT NULL,
    title TEXT,
    description TEXT,
    published_at TEXT,
    modified_at TEXT,
    withdrawn_at TEXT,
    source TEXT NOT NULL  -- NVD, CISA, OSV, vendor
);

-- CVE aliases
CREATE TABLE aliases (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vulnerability_id INTEGER NOT NULL,
    alias_id TEXT NOT NULL,
    source TEXT NOT NULL,
    FOREIGN KEY (vulnerability_id) REFERENCES vulnerabilities(id)
);

-- CWE mappings
CREATE TABLE cwe (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vulnerability_id INTEGER NOT NULL,
    cwe_id TEXT NOT NULL,
    FOREIGN KEY (vulnerability_id) REFERENCES vulnerabilities(id)
);

-- CVSS scores (support multiple versions)
CREATE TABLE cvss (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vulnerability_id INTEGER NOT NULL,
    version TEXT NOT NULL,  -- v2, v3.0, v3.1
    vector TEXT NOT NULL,
    base_score REAL,
    severity TEXT,
    FOREIGN KEY (vulnerability_id) REFERENCES vulnerabilities(id)
);

-- CPE dictionary
CREATE TABLE cpe (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    cpe_uri TEXT UNIQUE NOT NULL,
    vendor TEXT NOT NULL,
    product TEXT NOT NULL,
    version TEXT,
    part TEXT NOT NULL  -- a, h, o (application, hardware, operating system)
);

-- CVE-CPE matching
CREATE TABLE cpe_match (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vulnerability_id INTEGER NOT NULL,
    cpe_uri TEXT NOT NULL,
    vulnerable INTEGER DEFAULT 1,
    version_start TEXT,
    version_start_including INTEGER DEFAULT 0,
    version_end TEXT,
    version_end_including INTEGER DEFAULT 0,
    FOREIGN KEY (vulnerability_id) REFERENCES vulnerabilities(id)
);

-- Fixed version information
CREATE TABLE fixed_version (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vulnerability_id INTEGER NOT NULL,
    product TEXT NOT NULL,
    version TEXT NOT NULL,
    FOREIGN KEY (vulnerability_id) REFERENCES vulnerabilities(id)
);

-- EPSS scores
CREATE TABLE epss (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    cve_id TEXT NOT NULL,
    score REAL,
    percentile REAL,
    observed_at TEXT NOT NULL
);

-- CISA KEV catalog
CREATE TABLE kev (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    cve_id TEXT NOT NULL,
    date_added TEXT,
    due_date TEXT,
    vendor_project TEXT,
    product TEXT,
    vulnerability_name TEXT,
    required_action TEXT,
    known_ransomware_use INTEGER DEFAULT 0
);

-- External references
CREATE TABLE references (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    vulnerability_id INTEGER NOT NULL,
    url TEXT NOT NULL,
    source TEXT NOT NULL,
    reference_type TEXT,  -- advisory, patch, exploit, misc
    FOREIGN KEY (vulnerability_id) REFERENCES vulnerabilities(id)
);

-- Intelligence import metadata
CREATE TABLE intel_import (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source TEXT NOT NULL,
    source_version TEXT,
    imported_at TEXT NOT NULL,
    record_count INTEGER,
    checksum TEXT,
    status TEXT NOT NULL,  -- success, failed, rollback
    schema_version TEXT
);
```

#### 1.2 Encryption & Key Management

**Current State**:
- PBKDF2-SHA256 with 600,000 iterations
- Single key used for all databases
- Password complexity requirements enforced
- No key rotation mechanism

**Enhanced Security Model**:

```python
# Key hierarchy design
Master Password (user-provided)
    ↓ PBKDF2-SHA256 (600,000 iterations)
Key Encryption Key (KEK)
    ↓
Database Encryption Key (DEK) for security.db
    ↓
Intelligence Encryption Key (IEK) for vulnerability.db  
    ↓
Evidence Encryption Key (EEK) for evidence files
```

**Implementation Requirements**:
1. Separate encryption keys for each database
2. Key derivation with context-specific salts
3. Key rotation support (with re-encryption)
4. Secure key storage in memory only
5. Audit trail for key operations

#### 1.3 Observation Schema Definition

**Canonical Observation Schema**:

```python
OBSERVATION_SCHEMA = {
    "observation_id": "UUID",
    "scan_id": "UUID",
    "asset_id": "UUID (optional)",
    "service_id": "UUID (optional)",
    "scanner_id": "str (e.g., 'nmap', 'nuclei')",
    "scanner_version": "str",
    "observation_type": "enum",  # asset, port, service, technology, certificate, http, dns, configuration, vulnerability_candidate, vulnerability, secret, credential
    "title": "str",
    "raw_value": "dict (raw scanner output)",
    "normalized_value": "dict (normalized structure)",
    "confidence": "float (0.0-1.0)",
    "observed_at": "ISO8601 timestamp",
    "evidence_ids": "list[UUID]",
    "raw_output_hash": "SHA256",
    "parser_version": "str"
}
```

**Observation Types**:
- `asset` - Discovered host/IP/domain/subdomain
- `port` - Open port with protocol
- `service` - Service identification with version
- `technology` - Software/technology detection
- `certificate` - TLS/SSL certificate information
- `http` - HTTP response headers, status codes
- `dns` - DNS records and responses
- `configuration` - Configuration settings
- `vulnerability_candidate` - Potential vulnerability (unconfirmed)
- `vulnerability` - Confirmed vulnerability
- `secret` - Detected secret/credential
- `credential` - Valid credential pair
- `cloud_asset` - Cloud resource discovery
- `source_code` - Source code analysis results
- `dependency` - Software dependency information

#### 1.4 Finding Schema Definition

**Finding Schema**:

```python
FINDING_SCHEMA = {
    "finding_id": "UUID",
    "engagement_id": "UUID",
    "title": "str",
    "vulnerability_class": "str (e.g., 'SQL Injection', 'XSS')",
    "cwe_id": "str (e.g., 'CWE-89')",
    "cve_id": "str (e.g., 'CVE-2024-1234')",
    "asset_id": "UUID (optional)",
    "service_id": "UUID (optional)",
    "endpoint": "str (URL/path)",
    "parameter": "str (parameter name)",
    "severity": "enum (Critical, High, Medium, Low, Info)",
    "confidence": "float (0.0-1.0)",
    "evidence": "list[evidence_id]",
    "affected_observations": "list[observation_id]",
    "remediation": "str",
    "risk_score": "float (0-100)",
    "validation": "str (validation steps)",
    "status": "enum (open, in_progress, resolved, risk_accepted, false_positive)",
    "provenance": "dict (source information)"
}
```

#### 1.5 Scanner State Machine

**Explicit Scanner States**:

```python
SCANNER_STATES = {
    "NOT_STARTED": "Scanner not yet started",
    "BLOCKED": "Blocked by scope or policy",
    "DEPENDENCY_MISSING": "Required dependency unavailable",
    "STARTED": "Scanner process started",
    "RUNNING": "Scanner actively executing",
    "COMPLETED": "Scanner completed successfully",
    "COMPLETED_WITH_FINDINGS": "Completed with positive results",
    "COMPLETED_NO_FINDINGS": "Completed with no findings (must be verified)",
    "FAILED": "Scanner failed with error",
    "TIMEOUT": "Scanner exceeded time limit",
    "CANCELLED": "Scanner cancelled by user",
    "PARSE_FAILED": "Output parsing failed",
    "PARTIAL": "Partial results (incomplete scan)",
    "SKIPPED": "Skipped due to policy/conditions"
}
```

**State Transition Rules**:
- `NOT_STARTED` → `BLOCKED` | `DEPENDENCY_MISSING` | `STARTED`
- `STARTED` → `RUNNING` | `FAILED`
- `RUNNING` → `COMPLETED` | `COMPLETED_WITH_FINDINGS` | `COMPLETED_NO_FINDINGS` | `FAILED` | `TIMEOUT` | `CANCELLED` | `PARSE_FAILED` | `PARTIAL`
- No transitions from terminal states (`COMPLETED*`, `FAILED`, `TIMEOUT`, `CANCELLED`)

#### 1.6 Scanner Manifest Template

**Scanner Manifest Structure**:

```python
SCANNER_MANIFEST = {
    "id": "str (unique identifier)",
    "name": "str (display name)",
    "adapter_version": "str (semantic version)",
    "tool_version": "str (minimum required)",
    "category": "enum (recon, vuln_scan, network, web, code, cloud)",
    "input_type": "enum (host, domain, url, ip, cidr, file, code)",
    "output_format": "enum (json, xml, jsonl, txt, csv)",
    "activity_level": "enum (PASSIVE, LOW_IMPACT_ACTIVE, ACTIVE, INTRUSIVE, DESTRUCTIVE)",
    "requires_network": "bool",
    "external_services": "list[str]",
    "requires_credentials": "bool",
    "requires_root": "bool",
    "supports_offline": "bool",
    "default_timeout": "int (seconds)",
    "max_timeout": "int (seconds)",
    "max_concurrency": "int",
    "max_requests": "int",
    "max_output_bytes": "int",
    "supported_profiles": "list[str]",
    "parser": "str (parser module path)",
    "test_fixture": "str (test data path)",
    "dependencies": "list[str] (binary dependencies)"
}
```

#### 1.7 Implementation Tasks

**Database Migration**:
1. Create `database/schema/` directory with schema definition files
2. Write SQL migration scripts for security.db
3. Write SQL migration scripts for vulnerability.db
4. Create database initialization module
5. Implement backup/restore for current databases
6. Add schema version tracking

**Security Enhancement**:
1. Implement enhanced key hierarchy
2. Add key rotation functionality
3. Update encryption_manager.py with new key management
4. Add encryption key audit logging
5. Implement secure memory cleanup for keys

**Data Contracts**:
1. Create `core/observation.py` with observation schema and validation
2. Create `core/finding.py` with finding schema and validation
3. Create `core/scanner_manifest.py` with manifest template
4. Implement Pydantic models for schema validation
5. Add schema version compatibility checks

**State Machine**:
1. Create `core/state_machine.py` with scanner state definitions
2. Implement state transition validation
3. Add state transition audit logging
4. Create state machine tests

**Documentation**:
1. Create `docs/architecture/database.md` with detailed schema documentation
2. Create `docs/architecture/security.md` with encryption model
3. Create `docs/contracts/observation_schema.md`
4. Create `docs/contracts/finding_schema.md`
5. Create `docs/contracts/scanner_manifest.md`

**Testing**:
1. Add database schema tests
2. Add encryption/decryption tests
3. Add schema validation tests
4. Add state machine transition tests
5. Add key rotation tests

**Acceptance Criteria**:
- Application starts with new database schema
- Login works with enhanced encryption
- Encrypted DB opens only after key initialization
- Schema validation enforced for all data operations
- State machine prevents invalid transitions
- Test suite passes with new components
- Baseline regression report exists

**Git Commit**:
```
git add .
git commit -m "phase-1: establish security storage and data contracts"
```

### PHASE 2 — ENGAGEMENT, SCOPE & SCAN PLANNER

**Objective**: Make scope the authoritative security boundary and implement intelligent scan planning.

#### 2.1 Engagement Model

**Engagement Entity**:
```python
ENGAGEMENT_SCHEMA = {
    "engagement_id": "UUID",
    "name": "str",
    "description": "str",
    "start_date": "ISO8601",
    "end_date": "ISO8601",
    "status": "enum (active, completed, archived)",
    "created_by": "user_id",
    "created_at": "ISO8601",
    "team_members": "list[user_id]"
}
```

**Engagement Features**:
- Project-based organization of scans
- Team assignment (for future multi-user)
- Time-based engagement windows
- Status tracking and archival
- Engagement-level reporting

#### 2.2 Scope Engine

**Scope Rule Types**:
- `domain` - Domain rules (example.com, *.example.com)
- `subdomain` - Subdomain rules (api.example.com)
- `ip` - Specific IP addresses
- `cidr` - IP ranges (192.168.1.0/24)
- `url` - URL patterns (https://example.com/api/*)
- `port` - Port restrictions (80, 443, 8080-8090)

**Scope Rule Actions**:
- `allow` - Explicitly permit
- `deny` - Explicitly deny
- `log_only` - Log but allow (monitoring mode)

**Scope Rule Priority**:
- Higher priority = evaluated first
- Default priority = 100
- Allow explicit priority ordering

**Scope Engine Implementation**:

```python
class ScopeEngine:
    def __init__(self, engagement_id):
        self.engagement_id = engagement_id
        self.rules = self._load_scope_rules()
    
    def _load_scope_rules(self):
        """Load and cache scope rules for engagement."""
        pass
    
    def is_allowed(self, target, scanner_activity):
        """
        Check if target is allowed for given scanner activity.
        
        Args:
            target: str (URL, IP, domain)
            scanner_activity: enum (PASSIVE, LOW_IMPACT_ACTIVE, ACTIVE, INTRUSIVE, DESTRUCTIVE)
        
        Returns:
            tuple (allowed: bool, reason: str)
        """
        pass
    
    def check_redirect(self, original_target, redirect_target):
        """
        Check if redirect is within scope.
        
        Returns:
            tuple (allowed: bool, reason: str)
        """
        pass
    
    def add_rule(self, rule_type, rule_value, action, priority=100):
        """Add new scope rule."""
        pass
    
    def remove_rule(self, rule_id):
        """Remove scope rule."""
        pass
```

**Scope Validation Features**:
- CIDR matching for IP ranges
- Wildcard matching for domains
- Regex support for URL patterns
- Port range validation
- Activity level enforcement
- Redirect chain tracking
- Scope violation logging

#### 2.3 Scan Policy

**Scan Policy Definition**:

```python
SCAN_POLICY_SCHEMA = {
    "policy_id": "UUID",
    "engagement_id": "UUID",
    "name": "str",
    "scanner_allowlist": "list[str]",
    "scanner_denylist": "list[str]",
    "activity_level_limit": "enum (PASSIVE, LOW_IMPACT_ACTIVE, ACTIVE, INTRUSIVE, DESTRUCTIVE)",
    "rate_limits": {
        "requests_per_second": "int",
        "concurrent_scanners": "int"
    },
    "time_windows": {
        "allowed_hours": "list[int]",  # 0-23
        "allowed_days": "list[int]"   # 0-6 (Sun-Sat)
    },
    "max_duration": "int (seconds)",
    "auto_approve_scope": "bool"
}
```

**Policy Enforcement**:
- Scanner allowlist/denylist
- Activity level restrictions
- Rate limiting per target
- Time window enforcement
- Maximum scan duration
- Scope auto-approval

#### 2.4 Authorization Records

**Authorization Tracking**:

```python
AUTHORIZATION_SCHEMA = {
    "auth_id": "UUID",
    "engagement_id": "UUID",
    "target": "str",
    "authorized_by": "user_id",
    "authorized_at": "ISO8601",
    "expires_at": "ISO8601",
    "scope": "str",
    "limitations": "list[str]",
    "status": "enum (active, expired, revoked)"
}
```

**Authorization Features**:
- Per-target authorization tracking
- Expiration handling
- Limitation recording
- Revocation support
- Audit trail

#### 2.5 Scan Planner

**Planning Pipeline**:

```
Target Input
    ↓
Scope Engine Validation
    ↓
Asset Discovery Plan (Nmap, Subfinder, etc.)
    ↓
Enumeration Plan (HTTPx, WhatWeb, etc.)
    ↓
Technology Detection Plan (Tech Fingerprint, etc.)
    ↓
CVE Matching Plan (offline intelligence)
    ↓
Scanner Execution Plan (vulnerability scanners)
    ↓
Final Plan with Dependencies
```

**Planner Implementation**:

```python
class ScanPlanner:
    def __init__(self, engagement_id, target, scan_policy):
        self.engagement_id = engagement_id
        self.target = target
        self.policy = scan_policy
        self.scope_engine = ScopeEngine(engagement_id)
    
    def create_plan(self):
        """Create comprehensive scan plan."""
        # 1. Validate target against scope
        allowed, reason = self.scope_engine.is_allowed(self.target, "ACTIVE")
        if not allowed:
            raise ScopeViolationError(reason)
        
        # 2. Asset discovery phase
        asset_plan = self._plan_asset_discovery()
        
        # 3. Enumeration phase
        enum_plan = self._plan_enumeration()
        
        # 4. Technology detection
        tech_plan = self._plan_technology_detection()
        
        # 5. CVE matching
        cve_plan = self._plan_cve_matching()
        
        # 6. Vulnerability scanning
        vuln_plan = self._plan_vulnerability_scanning()
        
        # 7. Build dependency graph
        final_plan = self._build_dependency_graph([
            asset_plan, enum_plan, tech_plan, cve_plan, vuln_plan
        ])
        
        return final_plan
    
    def _plan_asset_discovery(self):
        """Plan asset discovery scanners."""
        pass
    
    def _plan_enumeration(self):
        """Plan enumeration scanners."""
        pass
    
    def _plan_technology_detection(self):
        """Plan technology detection."""
        pass
    
    def _plan_cve_matching(self):
        """Plan CVE intelligence matching."""
        pass
    
    def _plan_vulnerability_scanning(self):
        """Plan vulnerability scanners based on policy."""
        pass
    
    def _build_dependency_graph(self, phases):
        """Build dependency-resolved execution graph."""
        pass
```

**Planner Features**:
- Scope-aware planning
- Policy-compliant scanner selection
- Dependency resolution
- Phase-based organization
- Resource estimation
- Timeline prediction

#### 2.6 Implementation Tasks

**Engagement System**:
1. Create engagement management module
2. Implement engagement CRUD operations
3. Add engagement UI components (after backend stable)
4. Create engagement-based reporting

**Scope Engine**:
1. Implement scope rule parser
2. Create CIDR matching logic
3. Add wildcard domain matching
4. Implement regex URL pattern matching
5. Build scope validation engine
6. Add redirect chain tracking
7. Create scope violation logging

**Scan Policy**:
1. Implement policy definition module
2. Create policy enforcement engine
3. Add rate limiting implementation
4. Implement time window validation
5. Build policy evaluation system

**Authorization**:
1. Create authorization tracking module
2. Implement authorization validation
3. Add expiration handling
4. Create authorization revocation
5. Build authorization audit trail

**Scan Planner**:
1. Implement scan planner core
2. Create asset discovery planning
3. Build enumeration planning
4. Implement technology detection planning
5. Add CVE matching planning
6. Create vulnerability scanning planning
7. Build dependency graph resolver
8. Add resource estimation
9. Implement timeline prediction

**Testing**:
1. Add scope engine tests (CIDR, wildcard, regex)
2. Add policy enforcement tests
3. Add authorization tests
4. Add planner unit tests
5. Add integration tests for planning pipeline

**Documentation**:
1. Create `docs/architecture/scope_engine.md`
2. Create `docs/architecture/scan_planner.md`
3. Create `docs/engagement_model.md`
4. Create scope rule syntax guide
5. Create policy definition guide

**Acceptance Criteria**:
- Out-of-scope targets are blocked
- Redirects cannot silently escape scope
- Private/internal IP ranges handled correctly
- Scanner allowlist/denylist enforced
- Activity level restrictions work
- Rate limiting prevents overload
- Time windows respected
- Planner produces valid dependency graphs
- Resource estimates accurate

**Git Commit**:
```
git add .
git commit -m "phase-2: implement engagement scope and scan planner"
```

### PHASE 3 — SCANNER EXECUTION FRAMEWORK

**Objective**: Redesign scanner execution with proper verification, process management, and observation-based output.

#### 3.1 Scanner Adapter Framework

**Scanner Adapter Interface**:

```python
class ScannerAdapter(ABC):
    """Base class for all scanner adapters."""
    
    @abstractmethod
    def get_manifest(self) -> dict:
        """Return scanner manifest."""
        pass
    
    @abstractmethod
    def verify_binary(self) -> tuple[bool, str]:
        """Verify binary availability and version."""
        pass
    
    @abstractmethod
    def prepare_execution(self, target: str, config: dict) -> dict:
        """Prepare execution parameters."""
        pass
    
    @abstractmethod
    def execute(self, target: str, config: dict) -> dict:
        """Execute scanner and return raw result."""
        pass
    
    @abstractmethod
    def parse_output(self, raw_output: str) -> list[dict]:
        """Parse raw output into observations."""
        pass
    
    @abstractmethod
    def cleanup(self, workspace: str):
        """Clean up temporary workspace."""
        pass
```

**Adapter Registry**:

```python
class AdapterRegistry:
    def __init__(self):
        self.adapters = {}
    
    def register(self, adapter: ScannerAdapter):
        """Register a scanner adapter."""
        manifest = adapter.get_manifest()
        self.adapters[manifest['id']] = adapter
    
    def get_adapter(self, scanner_id: str) -> ScannerAdapter:
        """Get adapter by scanner ID."""
        return self.adapters.get(scanner_id)
    
    def list_adapters(self) -> list[dict]:
        """List all registered adapters."""
        return [adapter.get_manifest() for adapter in self.adapters.values()]
```

#### 3.2 Binary Verification

**Binary Verification Checklist**:

```python
class BinaryVerifier:
    def verify(self, binary_name: str, required_version: str = None) -> dict:
        """
        Comprehensive binary verification.
        
        Returns:
            {
                "available": bool,
                "path": str,
                "version": str,
                "compatible": bool,
                "checksum": str,
                "installation_method": str,
                "offline_capable": bool
            }
        """
        pass
    
    def check_installation(self, binary_name: str) -> tuple[bool, str]:
        """Check if binary is installed."""
        pass
    
    def get_version(self, binary_path: str) -> str:
        """Get binary version."""
        pass
    
    def verify_checksum(self, binary_path: str, expected_checksum: str) -> bool:
        """Verify binary checksum."""
        pass
    
    def check_offline_capability(self, binary_name: str) -> bool:
        """Check if scanner works offline."""
        pass
```

#### 3.3 Process Verification

**Process Tracker**:

```python
class ProcessTracker:
    def __init__(self, scan_id: str):
        self.scan_id = scan_id
        self.processes = {}
    
    def start_process(self, scanner_name: str, command: list, workspace: str) -> str:
        """Start and track a subprocess."""
        pass
    
    def monitor_process(self, pid: str) -> dict:
        """Monitor process status."""
        pass
    
    def terminate_process(self, pid: str, force: bool = False) -> bool:
        """Terminate process and process tree."""
        pass
    
    def get_process_info(self, pid: str) -> dict:
        """Get detailed process information."""
        pass
```

**Process Information Recorded**:
- Process ID (PID)
- Parent process ID
- Command line (sanitized)
- Start time
- Exit code
- Exit time
- CPU usage
- Memory usage
- Stdout/stderr capture status
- Output truncation status
- Child process status

#### 3.4 Enhanced Timeout Management

**Timeout Configuration**:

```python
TIMEOUT_CONFIG = {
    "hard_timeout": "int (absolute maximum)",
    "soft_timeout": "int (graceful shutdown)",
    "process_tree_kill": "bool",
    "cleanup_timeout": "int",
    "timeout_reason": "str"
}
```

**Timeout Handler**:

```python
class TimeoutHandler:
    def __init__(self, hard_timeout: int, soft_timeout: int = None):
        self.hard_timeout = hard_timeout
        self.soft_timeout = soft_timeout
        self.start_time = None
    
    def start_timer(self):
        """Start timeout timer."""
        pass
    
    def check_timeout(self) -> tuple[bool, str]:
        """Check if timeout exceeded."""
        pass
    
    def handle_timeout(self, process) -> str:
        """Handle timeout with graceful shutdown."""
        pass
```

#### 3.5 Retry Logic

**Retry Policy**:

```python
RETRY_POLICY = {
    "max_retries": "int",
    "retryable_errors": "list[str]",
    "non_retryable_errors": "list[str]",
    "backoff_strategy": "enum (fixed, exponential, linear)",
    "initial_delay": "int",
    "max_delay": "int"
}
```

**Retry Handler**:

```python
class RetryHandler:
    def __init__(self, policy: dict):
        self.policy = policy
        self.attempt_count = 0
    
    def should_retry(self, error: str, exit_code: int) -> bool:
        """Determine if error is retryable."""
        pass
    
    def get_delay(self) -> int:
        """Calculate retry delay based on strategy."""
        pass
    
    def record_attempt(self, success: bool):
        """Record retry attempt."""
        pass
```

**Retryable Errors**:
- Transient tool startup failure
- Temporary resource failure
- Transient network failure (where policy permits)
- Timeout with partial results

**Non-Retryable Errors**:
- Invalid target
- Scope violation
- Authorization failure
- Malformed configuration
- Parser bug
- Unsupported platform

#### 3.6 Scope Integration

**Pre-Execution Scope Check**:

```python
def check_scope_before_execution(target: str, scanner_id: str, scope_engine: ScopeEngine) -> tuple[bool, str]:
    """
    Check scope before every scanner execution.
    
    Returns:
        (allowed: bool, reason: str)
    """
    manifest = get_scanner_manifest(scanner_id)
    activity_level = manifest['activity_level']
    
    return scope_engine.is_allowed(target, activity_level)
```

#### 3.7 Activity Classification

**Activity Level Enforcement**:

```python
ACTIVITY_LEVELS = {
    "PASSIVE": "No interaction with target, purely information gathering",
    "LOW_IMPACT_ACTIVE": "Minimal interaction, low risk of disruption",
    "ACTIVE": "Active probing, moderate risk",
    "INTRUSIVE": "Potential system impact, high risk",
    "DESTRUCTIVE": "May cause damage or disruption"
}

def enforce_activity_level(scanner_id: str, allowed_level: str) -> bool:
    """Enforce activity level restrictions."""
    manifest = get_scanner_manifest(scanner_id)
    scanner_level = manifest['activity_level']
    
    level_hierarchy = {
        "PASSIVE": 0,
        "LOW_IMPACT_ACTIVE": 1,
        "ACTIVE": 2,
        "INTRUSIVE": 3,
        "DESTRUCTIVE": 4
    }
    
    return level_hierarchy[scanner_level] <= level_hierarchy[allowed_level]
```

#### 3.8 Resource Management

**Resource Limits**:

```python
RESOURCE_LIMITS = {
    "cpu_expectation": "float (0.0-1.0)",
    "memory_limit_mb": "int",
    "concurrency": "int",
    "request_rate": "int (requests/second)",
    "output_size_limit_bytes": "int",
    "disk_usage_limit_mb": "int",
    "timeout": "int",
    "network_policy": "str"
}
```

**Resource Monitor**:

```python
class ResourceMonitor:
    def monitor_resources(self, pid: str) -> dict:
        """Monitor resource usage for process."""
        pass
    
    def check_limits(self, usage: dict, limits: dict) -> bool:
        """Check if resource limits exceeded."""
        pass
    
    def enforce_limits(self, pid: str, limits: dict):
        """Enforce resource limits."""
        pass
```

#### 3.9 Provenance Tracking

**Execution Provenance**:

```python
PROVENANCE_SCHEMA = {
    "scanner_id": "str",
    "scanner_name": "str",
    "scanner_version": "str",
    "adapter_version": "str",
    "command_identifier": "str",
    "configuration_hash": "str",
    "target": "str",
    "scope": "str",
    "start_time": "ISO8601",
    "end_time": "ISO8601",
    "exit_code": "int",
    "status": "str",
    "raw_output_hash": "str",
    "parser_version": "str"
}
```

**Provenance Recorder**:

```python
class ProvenanceRecorder:
    def record_execution(self, execution_data: dict):
        """Record execution provenance."""
        pass
    
    def get_provenance(self, scan_id: str, scanner_name: str) -> dict:
        """Retrieve execution provenance."""
        pass
```

#### 3.10 Observation Parser Framework

**Parser Interface**:

```python
class ObservationParser(ABC):
    @abstractmethod
    def parse(self, raw_output: str, scanner_context: dict) -> list[dict]:
        """Parse raw output into observations."""
        pass
    
    @abstractmethod
    def validate_observation(self, observation: dict) -> bool:
        """Validate observation against schema."""
        pass
    
    @abstractmethod
    def normalize_observation(self, observation: dict) -> dict:
        """Normalize observation to standard format."""
        pass
```

**Parser Registry**:

```python
class ParserRegistry:
    def register_parser(self, scanner_id: str, parser: ObservationParser):
        """Register parser for scanner."""
        pass
    
    def get_parser(self, scanner_id: str) -> ObservationParser:
        """Get parser for scanner."""
        pass
```

#### 3.11 Implementation Tasks

**Adapter Framework**:
1. Create scanner adapter base class
2. Implement adapter registry
3. Create adapter interface documentation
4. Build adapter testing framework

**Binary Verification**:
1. Implement binary verifier
2. Add version checking logic
3. Implement checksum verification
4. Add offline capability detection
5. Create binary cache

**Process Management**:
1. Implement process tracker
2. Add process tree handling
3. Create stdout/stderr capture
4. Implement output truncation handling
5. Add child process monitoring

**Timeout Management**:
1. Implement timeout handler
2. Add soft timeout support
3. Create process tree kill
4. Implement cleanup procedures
5. Add timeout reason recording

**Retry Logic**:
1. Implement retry handler
2. Add retry policy configuration
3. Create backoff strategies
4. Implement retry attempt logging
5. Add retry success/failure tracking

**Scope Integration**:
1. Integrate scope engine with execution
2. Add pre-execution scope checks
3. Implement redirect scope validation
4. Create scope violation handling

**Activity Classification**:
1. Implement activity level enforcement
2. Add activity level to manifests
3. Create activity level validation
4. Build activity-based filtering

**Resource Management**:
1. Implement resource monitor
2. Add resource limit enforcement
3. Create resource usage tracking
4. Implement resource-based scheduling

**Provenance Tracking**:
1. Implement provenance recorder
2. Add execution metadata capture
3. Create provenance query interface
4. Build provenance audit trail

**Observation Parsing**:
1. Create parser base class
2. Implement parser registry
3. Build observation validation
4. Add observation normalization
5. Create parser testing framework

**Testing**:
1. Add adapter framework tests
2. Add binary verification tests
3. Add process management tests
4. Add timeout handling tests
5. Add retry logic tests
6. Add scope integration tests
7. Add resource management tests
8. Add provenance tracking tests
9. Add parser framework tests
10. Add integration tests for full execution pipeline

**Documentation**:
1. Create `docs/architecture/scanner_adapter.md`
2. Create `docs/architecture/execution_framework.md`
3. Create adapter development guide
4. Create parser development guide
5. Create testing guide for adapters

**Acceptance Criteria**:
- Binary verification works for all scanners
- Process tracking captures all required information
- Timeout handling prevents hanging scanners
- Retry logic handles transient failures appropriately
- Scope checks prevent unauthorized scanning
- Activity levels are properly enforced
- Resource limits prevent system overload
- Provenance tracking provides complete audit trail
- Observation parsers produce valid observations
- Execution framework handles all scanner states correctly

**Git Commit**:
```
git add .
git commit -m "phase-3: implement scanner execution framework"
```

### PHASE 4 — VULNERABILITY INTELLIGENCE SYSTEM

**Objective**: Create dedicated vulnerability intelligence subsystem with offline capability and proper CVE/CPE matching.

#### 4.1 Intelligence Source Adapters

**Supported Sources**:
- NVD CVE/CPE data
- CISA KEV catalog
- EPSS scores (FIRST.org)
- OSV (Open Source Vulnerabilities)
- GitHub Security Advisories
- Vendor advisories (future extensibility)

**Adapter Interface**:

```python
class IntelAdapter(ABC):
    @abstractmethod
    def fetch(self, params: dict) -> dict:
        """Fetch intelligence from source."""
        pass
    
    @abstractmethod
    def parse(self, raw_data: dict) -> list[dict]:
        """Parse raw data into normalized format."""
        pass
    
    @abstractmethod
    def validate(self, parsed_data: list[dict]) -> bool:
        """Validate parsed data integrity."""
        pass
    
    @abstractmethod
    def get_source_metadata(self) -> dict:
        """Return source metadata (version, timestamp, etc.)."""
        pass
```

#### 4.2 Intelligence Lifecycle

**Lifecycle Pipeline**:

```
SOURCE
    ↓
DOWNLOAD (with retry logic)
    ↓
VERIFY (checksum, signature)
    ↓
PARSE (normalize to internal format)
    ↓
VALIDATE (schema validation)
    ↓
INDEX (create search indexes)
    ↓
ATOMIC IMPORT (staging → production swap)
    ↓
LOCAL QUERY (offline capability)
```

**Staging Database**:
- Download to staging vulnerability.db.staging
- Validate complete import
- Verify record counts and checksums
- Atomic swap with production database
- Rollback capability on failure

#### 4.3 Offline Matching Engine

**Matching Pipeline**:

```
Scanner Observation
    ↓
Vendor/Product/Version Normalization
    ↓
CPE Candidate Generation
    ↓
CPE Applicability Check
    ↓
Version Range Evaluation
    ↓
CVE Candidate Selection
    ↓
Confidence Scoring
```

**Matching States**:

```python
MATCHING_STATES = {
    "NO_MATCH": "No applicable CVEs found",
    "CANDIDATE": "Potential match, low confidence",
    "LIKELY_AFFECTED": "High probability match",
    "AFFECTED_BY_VERSION": "Version range indicates vulnerability",
    "CONFIRMED_BY_EVIDENCE": "Active scanner evidence confirms vulnerability",
    "NOT_AFFECTED": "Explicitly not affected",
    "UNKNOWN": "Unable to determine"
}
```

**CPE Normalization**:
- Extract vendor/product/version from observations
- Normalize to standard CPE format
- Handle version edge cases (1.0 vs 1.0.0)
- Generate multiple CPE candidates for fuzzy matching

**Version Range Evaluation**:
- Parse CPE version ranges (start_including, end_including)
- Compare detected versions against ranges
- Handle pre-release versions
- Support semantic version comparison

#### 4.4 Implementation Tasks

**Intelligence Adapters**:
1. Create NVD adapter with full/incremental sync
2. Implement CISA KEV adapter
3. Build EPSS adapter with incremental updates
4. Add OSV adapter for open-source projects
5. Create GitHub Security Advisory adapter
6. Implement adapter registry

**Lifecycle Management**:
1. Create staging database management
2. Implement atomic import/swap logic
3. Add rollback functionality
4. Build import verification system
5. Create checksum validation
6. Implement signature verification (where available)

**Offline Matching**:
1. Build CPE normalization engine
2. Implement version range parser
3. Create semantic version comparator
4. Build confidence scoring algorithm
5. Implement matching state machine
6. Add caching for repeated matches

**Database Schema**:
1. Implement vulnerability.db schema
2. Create proper indexes for performance
3. Add FTS (Full-Text Search) for CVE descriptions
4. Implement version tracking for schema migrations
5. Create import metadata tables

**Testing**:
1. Add adapter tests for each source
2. Test lifecycle management (staging, import, rollback)
3. Add offline matching tests with known CVEs
4. Test version range evaluation edge cases
5. Create performance tests for large CVE databases

**Documentation**:
1. Create `docs/architecture/vulnerability_intelligence.md`
2. Document adapter development process
3. Create offline matching algorithm guide
4. Document CPE normalization rules
5. Create intelligence update procedures

**Acceptance Criteria**:
- All intelligence sources sync correctly
- Staging database validates before swap
- Atomic import prevents partial updates
- Rollback works on import failure
- Offline matching produces accurate results
- Version range evaluation handles edge cases
- Confidence scoring reflects reality
- System works completely offline after initial sync

**Git Commit**:
```
git add .
git commit -m "phase-4: implement vulnerability intelligence system"
```

### PHASE 5 — OBSERVATION MODEL & EVIDENCE ENGINE

**Objective**: Implement canonical observation schema and dedicated evidence abstraction.

#### 5.1 Observation Implementation

**Observation Model**:

```python
class Observation:
    def __init__(self, data: dict):
        self.validate(data)
        self.data = data
        self.immutable = True
    
    def validate(self, data: dict):
        """Validate against observation schema."""
        pass
    
    def to_dict(self) -> dict:
        """Return immutable copy."""
        return self.data.copy()
    
    def add_evidence(self, evidence_id: str):
        """Add evidence reference (if not yet stored)."""
        if not self.immutable:
            self.data['evidence_ids'].append(evidence_id)
```

**Observation Types Implementation**:
- Create type-specific observation classes
- Implement type-specific validation
- Add type-specific normalization
- Create observation factory

#### 5.2 Evidence Engine

**Evidence Types**:
- `raw_scanner_output` - Complete scanner stdout/stderr
- `http_request` - HTTP request details
- `http_response` - HTTP response headers/body
- `headers` - HTTP headers collection
- `screenshot` - Visual evidence capture
- `nmap_xml` - Nmap XML output
- `nuclei_json` - Nuclei JSONL output
- `json` - Generic JSON data
- `xml` - Generic XML data
- `sarif` - SARIF format
- `text` - Plain text evidence
- `certificate` - TLS/SSL certificate
- `configuration` - Configuration files
- `source_file` - Source code artifacts
- `command_output` - Command execution results

**Evidence Storage**:

```python
class EvidenceStore:
    def __init__(self, base_path: str, encryption_key: str):
        self.base_path = base_path
        self.encryption_key = encryption_key
    
    def store_evidence(self, evidence_type: str, data: bytes, metadata: dict) -> str:
        """
        Store evidence with encryption.
        
        Returns:
            evidence_id (UUID)
        """
        evidence_id = str(uuid.uuid4())
        encrypted_data = self._encrypt(data)
        
        path = self._get_evidence_path(evidence_id, evidence_type, metadata)
        os.makedirs(os.path.dirname(path), exist_ok=True)
        
        with open(path, 'wb') as f:
            f.write(encrypted_data)
        
        self._store_metadata(evidence_id, metadata)
        return evidence_id
    
    def retrieve_evidence(self, evidence_id: str) -> tuple[bytes, dict]:
        """Retrieve and decrypt evidence."""
        metadata = self._get_metadata(evidence_id)
        path = self._get_evidence_path(evidence_id, metadata['type'], metadata)
        
        with open(path, 'rb') as f:
            encrypted_data = f.read()
        
        decrypted_data = self._decrypt(encrypted_data)
        return decrypted_data, metadata
```

**Evidence Structure**:
```
data/evidence/<engagement_id>/<scan_id>/<evidence_id>/
├── evidence.enc              # Encrypted evidence data
├── metadata.json             # Evidence metadata (unencrypted)
└── checksum.txt              # SHA-256 checksum
```

#### 5.3 Evidence Provenance

**Provenance Tracking**:
- Creation timestamp
- Source scanner
- Scanner version
- Observation ID
- Finding ID (if associated)
- Chain of custody
- Access logs

**Evidence Schema**:

```python
EVIDENCE_METADATA = {
    "evidence_id": "UUID",
    "observation_id": "UUID",
    "finding_id": "UUID (optional)",
    "type": "str",
    "size_bytes": "int",
    "sha256": "str",
    "created_at": "ISO8601",
    "scanner_source": "str",
    "scanner_version": "str",
    "encryption_version": "str",
    "storage_path": "str",
    "access_count": "int",
    "last_accessed": "ISO8601"
}
```

#### 5.4 Temporary Workspace Management

**Workspace Lifecycle**:
```
Create Workspace
    ↓
Execute Scanner
    ↓
Capture Raw Output
    ↓
Parse to Observations
    ↓
Store Evidence
    ↓
Archive Important Evidence
    ↓
Cleanup Temporary Files
```

**Workspace Structure**:
```
data/work/<scan_id>/
├── scanner_outputs/         # Raw scanner outputs
├── parsed_data/             # Intermediate parsed data
├── evidence/                # Evidence candidates
└── logs/                    # Scanner-specific logs
```

#### 5.5 Implementation Tasks

**Observation Model**:
1. Implement core Observation class
2. Create type-specific observation subclasses
3. Build observation validation framework
4. Implement observation factory
5. Add observation serialization/deserialization

**Evidence Engine**:
1. Create EvidenceStore class
2. Implement encryption/decryption
3. Build evidence metadata management
4. Add evidence type handlers
5. Implement evidence retrieval
6. Create evidence cleanup

**Provenance Tracking**:
1. Implement evidence provenance recorder
2. Add access logging
3. Create chain of custody tracking
4. Build provenance query interface

**Workspace Management**:
1. Create workspace manager
2. Implement workspace lifecycle
3. Add automatic cleanup
4. Create workspace isolation
5. Implement evidence archival

**Database Integration**:
1. Update observation table implementation
2. Add evidence metadata table
3. Create evidence-evidence_id mappings
4. Implement observation-evidence relationships

**Testing**:
1. Add observation model tests
2. Test evidence encryption/decryption
3. Add evidence lifecycle tests
4. Test workspace management
5. Create performance tests for large evidence
6. Test provenance tracking

**Documentation**:
1. Create `docs/architecture/evidence_engine.md`
2. Document observation model
3. Create evidence storage guide
4. Document workspace lifecycle
5. Create evidence API reference

**Acceptance Criteria**:
- Observations are immutable after creation
- Evidence encryption works correctly
- Evidence retrieval produces original data
- Workspace cleanup prevents disk bloat
- Provenance tracking is complete
- Evidence types are properly handled
- Large evidence files don't impact database performance
- Temporary files are cleaned up appropriately

**Git Commit**:
```
git add .
git commit -m "phase-5: implement observation model and evidence engine"
```

### PHASE 6 — SCAN ORCHESTRATOR REDESIGN

**Objective**: Separate scanner orchestration responsibilities into distinct components.

#### 6.1 Component Separation

**New Architecture**:

```
Scope Engine
    ↓
Planner
    ↓
Scheduler
    ↓
Execution Sandbox
    ↓
Scanner Adapter
    ↓
Observation Parser
    ↓
Evidence Store
    ↓
Finding Engine
```

**Component Responsibilities**:
- **Scope Engine**: Authorization and boundary enforcement
- **Planner**: Dependency resolution and execution planning
- **Scheduler**: Coordinated execution and resource management
- **Execution Sandbox**: Process isolation and control
- **Scanner Adapter**: Tool interface and execution
- **Observation Parser**: Output normalization
- **Evidence Store**: Raw output preservation
- **Finding Engine**: Correlation and deduplication

#### 6.2 Scheduler Implementation

**Scheduler Features**:
- DAG-based execution (preserving existing Kahn's algorithm)
- Concurrent execution with limits
- Pause/resume capability
- Failure handling and recovery
- Resource-aware scheduling
- Priority queue management

**Scheduler Interface**:

```python
class ScanScheduler:
    def __init__(self, max_concurrent: int = 5):
        self.max_concurrent = max_concurrent
        self.active_scans = {}
        self.scan_queue = []
    
    def schedule_scan(self, execution_plan: dict) -> str:
        """Schedule a scan for execution."""
        pass
    
    def pause_scan(self, scan_id: str):
        """Pause an active scan."""
        pass
    
    def resume_scan(self, scan_id: str):
        """Resume a paused scan."""
        pass
    
    def cancel_scan(self, scan_id: str):
        """Cancel a scan execution."""
        pass
    
    def get_scan_status(self, scan_id: str) -> dict:
        """Get current scan status."""
        pass
```

#### 6.3 Execution Sandbox

**Sandbox Features**:
- Process isolation
- Resource limits
- Network policy enforcement
- Filesystem isolation
- Time limits
- Cleanup procedures

**Sandbox Implementation**:

```python
class ExecutionSandbox:
    def __init__(self, workspace: str, limits: dict):
        self.workspace = workspace
        self.limits = limits
    
    def execute_in_sandbox(self, command: list, environment: dict) -> dict:
        """Execute command within sandbox constraints."""
        pass
    
    def enforce_resource_limits(self, pid: int):
        """Enforce CPU/memory limits."""
        pass
    
    def cleanup(self):
        """Clean up sandbox resources."""
        pass
```

#### 6.4 Finding Engine

**Finding Correlation**:
- Observation aggregation
- CVE intelligence matching
- Confidence calculation
- Severity normalization
- Risk scoring integration

**Finding Engine Interface**:

```python
class FindingEngine:
    def __init__(self, vulnerability_db_path: str):
        self.vuln_db = vulnerability_db_path
    
    def correlate_findings(self, observations: list[dict]) -> list[dict]:
        """Correlate observations into findings."""
        pass
    
    def match_cve(self, observation: dict) -> list[dict]:
        """Match observation against CVE database."""
        pass
    
    def calculate_confidence(self, finding: dict) -> float:
        """Calculate finding confidence score."""
        pass
    
    def normalize_severity(self, finding: dict) -> str:
        """Normalize severity across sources."""
        pass
```

#### 6.5 Implementation Tasks

**Scheduler**:
1. Implement scan scheduler core
2. Add DAG execution with Kahn's algorithm
3. Implement concurrency control
4. Add pause/resume functionality
5. Create failure recovery logic
6. Build priority queue management

**Execution Sandbox**:
1. Create sandbox manager
2. Implement process isolation
3. Add resource limit enforcement
4. Build network policy control
5. Create filesystem isolation
6. Implement cleanup procedures

**Finding Engine**:
1. Implement finding correlation logic
2. Add CVE matching integration
3. Build confidence calculation
4. Implement severity normalization
5. Create risk scoring integration
6. Add finding deduplication

**Integration**:
1. Wire components together
2. Implement data flow between components
3. Add error handling and recovery
4. Create component health checks
5. Build performance monitoring

**Testing**:
1. Add scheduler unit tests
2. Test sandbox isolation
3. Add finding engine tests
4. Test component integration
5. Create end-to-end orchestration tests
6. Add performance tests

**Documentation**:
1. Create `docs/architecture/orchestrator.md`
2. Document component interfaces
3. Create scheduler configuration guide
4. Document sandbox policies
5. Create finding engine algorithm guide

**Acceptance Criteria**:
- Components are properly separated
- Scheduler executes DAGs correctly
- Sandbox provides isolation
- Finding engine produces accurate correlations
- Components communicate effectively
- Error handling prevents cascading failures
- Performance is acceptable
- System can handle concurrent scans

**Git Commit**:
```
git add .
git commit -m "phase-6: redesign scan orchestrator"
```

### PHASE 7 — DEDUPLICATION & FINDING MODEL

**Objective**: Replace destructive deduplication with evidence-preserving correlation.

#### 7.1 New Deduplication Strategy

**Fingerprint-Based Deduplication**:

```python
FINGERPRINT_COMPONENTS = {
    "asset": "str",
    "service": "str", 
    "endpoint": "str",
    "parameter": "str",
    "vulnerability_class": "str",
    "normalized_technology": "str",
    "evidence_signature": "str"
}
```

**Deduplication Process**:
1. Generate fingerprint for each finding
2. Exact fingerprint match = same finding
3. Fuzzy similarity = correlation signal
4. Preserve all source observations
5. Merge with evidence preservation

**Evidence-Preserving Merge**:

```python
def merge_findings(findings: list[dict]) -> dict:
    """
    Merge findings while preserving all source observations.
    
    Canonical Finding:
        ├── Observation A (Nuclei)
        ├── Observation B (Nikto)  
        └── Observation C (Custom)
    """
    canonical = findings[0].copy()
    canonical['affected_observations'] = []
    
    for finding in findings:
        canonical['affected_observations'].extend(finding['affected_observations'])
        canonical['scanner_sources'].add(finding['scanner_source'])
    
    return canonical
```

#### 7.2 Finding Model Enhancement

**Enhanced Finding Schema**:

```python
ENHANCED_FINDING_SCHEMA = {
    "finding_id": "UUID",
    "fingerprint": "str (SHA-256 of fingerprint components)",
    "engagement_id": "UUID",
    "title": "str",
    "vulnerability_class": "str",
    "cwe_id": "str",
    "cve_id": "list[str]",  # Multiple CVEs possible
    "asset_id": "UUID",
    "service_id": "UUID",
    "endpoint": "str",
    "parameter": "str",
    "severity": "str",
    "confidence": "float",
    "evidence": "list[UUID]",
    "affected_observations": "list[UUID]",  # Preserved source observations
    "scanner_sources": "set[str]",  # All scanners that reported this
    "remediation": "str",
    "risk_score": "float",
    "validation": "str",
    "status": "str",
    "provenance": "dict",
    "first_observed_at": "ISO8601",
    "last_observed_at": "ISO8601",
    "occurrence_count": "int"  # How many times this has been seen
}
```

#### 7.3 Implementation Tasks

**Deduplication Engine**:
1. Implement fingerprint generation
2. Create exact match deduplication
3. Add fuzzy similarity correlation
4. Build evidence-preserving merge
5. Implement occurrence tracking

**Finding Model**:
1. Update finding schema implementation
2. Add multi-CVE support
3. Implement occurrence counting
4. Create finding versioning
5. Add finding lifecycle management

**Integration**:
1. Integrate with finding engine
2. Update observation correlation
3. Add deduplication to scan pipeline
4. Create deduplication reporting
5. Build manual deduplication tools

**Testing**:
1. Add fingerprint generation tests
2. Test deduplication accuracy
3. Add evidence preservation tests
4. Test multi-CVE handling
5. Create performance tests

**Documentation**:
1. Create `docs/architecture/deduplication.md`
2. Document fingerprint algorithm
3. Create deduplication strategy guide
4. Document finding model changes
5. Create manual deduplication procedures

**Acceptance Criteria**:
- Fingerprint generation is consistent
- Exact matches are correctly identified
- Fuzzy correlation provides useful signals
- All source observations are preserved
- Evidence is never lost in deduplication
- Multi-CVE findings are handled correctly
- Occurrence tracking is accurate
- Performance is acceptable for large finding sets

**Git Commit**:
```
git add .
git commit -m "phase-7: implement evidence-preserving deduplication"
```

### PHASE 8 — NMAP SERVICE DISCOVERY INTEGRATION

**Objective**: Properly integrate Nmap as primary service discovery source, not generic scanner.

#### 8.1 Nmap Adapter Enhancement

**Special Nmap Treatment**:
- Asset discovery (hosts, IPs, domains)
- Service enumeration (ports, protocols)
- Technology detection (product, version)
- CPE extraction (for CVE matching)
- NSE script results (vulnerability observations)

**Enhanced Nmap Output Parsing**:

```python
class NmapParser(ObservationParser):
    def parse(self, raw_output: str, scanner_context: dict) -> list[dict]:
        """Parse Nmap XML into multiple observation types."""
        root = ET.fromstring(raw_output)
        observations = []
        
        # Asset observations
        for host in root.findall('.//host'):
            observations.extend(self._parse_asset(host))
        
        # Service observations
        for port in root.findall('.//port'):
            observations.extend(self._parse_service(port))
        
        # Technology observations
        for service in root.findall('.//service'):
            observations.extend(self._parse_technology(service))
        
        # CPE observations
        for cpe in root.findall('.//cpe'):
            observations.extend(self._parse_cpe(cpe))
        
        # NSE vulnerability observations
        for script in root.findall('.//script'):
            observations.extend(self._parse_nse_vuln(script))
        
        return observations
```

#### 8.2 CPE Extraction Pipeline

**CPE Extraction**:
- Extract CPE from Nmap service detection
- Normalize CPE format
- Generate CPE candidates
- Store in observations for CVE matching

**CPE Observation Schema**:

```python
CPE_OBSERVATION = {
    "observation_type": "cpe",
    "cpe_uri": "str",
    "vendor": "str",
    "product": "str",
    "version": "str",
    "part": "str",
    "confidence": "float",
    "source": "nmap_service_detection"
}
```

#### 8.3 Service Discovery Integration

**Pipeline Integration**:
```
Nmap Execution
    ↓
Asset Discovery (hosts, IPs)
    ↓
Service Enumeration (ports, protocols)
    ↓
Technology Detection (products, versions)
    ↓
CPE Extraction
    ↓
CVE Matching (using Phase 4 intelligence)
    ↓
Asset/Service Creation in database
```

#### 8.4 Implementation Tasks

**Nmap Adapter**:
1. Enhance Nmap adapter with special treatment
2. Implement multi-type observation parsing
3. Add CPE extraction logic
4. Create NSE script result parsing
5. Add service discovery optimization

**CPE Pipeline**:
1. Implement CPE extraction
2. Create CPE normalization
3. Build CPE candidate generation
4. Add CPE observation storage
5. Integrate with CVE matching

**Database Integration**:
1. Update asset creation from Nmap
2. Implement service population
3. Add CPE observation storage
4. Create asset-service relationships
5. Build technology tracking

**Testing**:
1. Add Nmap parsing tests
2. Test CPE extraction accuracy
3. Add service discovery integration tests
4. Test CVE matching with Nmap data
5. Create performance tests

**Documentation**:
1. Create `docs/architecture/nmap_integration.md`
2. Document Nmap special treatment
3. Create CPE extraction guide
4. Document service discovery pipeline
5. Create Nmap adapter development guide

**Acceptance Criteria**:
- Nmap produces multiple observation types
- Assets are correctly discovered
- Services are properly enumerated
- CPE extraction is accurate
- CVE matching works with Nmap data
- NSE results are captured
- Service discovery feeds technology detection
- Pipeline integration is seamless

**Git Commit**:
```
git add .
git commit -m "phase-8: integrate nmap as service discovery source"
```

### PHASE 9 — UI REFACTORING

**Objective**: Rebuild UI information architecture to reflect new security model.

#### 9.1 UI Architecture Redesign

**New Navigation Structure**:
- Dashboard
- Targets
- Engagements
- Scans
- Assets
- Services
- Technologies
- Vulnerabilities
- Findings
- Evidence
- Risk
- Remediation
- Reports
- CVE Intelligence
- System / Tools
- Audit
- Settings

#### 9.2 Dashboard Redesign

**New Dashboard Metrics**:
- Total Assets
- Internet-Facing Assets
- Open Findings
- Critical/High Findings
- Confirmed Vulnerabilities
- CVE Intelligence Age
- Last Successful Intelligence Update
- Active Scans
- Failed Scanners
- Risk Score
- Risk Trend

**Dashboard Implementation**:
- Real-time metric updates
- Risk trend visualization
- Active scan monitoring
- Intelligence health indicators
- System status overview

#### 9.3 Scan Screen Enhancement

**Enhanced Scan Display**:
- Engagement context
- Target and scope information
- Profile and policy details
- Real-time progress tracking
- Current phase and scanner
- Completed/failed/skipped scanners
- Findings discovered in real-time
- CVE candidates identified
- Confirmed findings count
- Evidence capture status

**Progress Calculation**:
- Based on actual scanner states
- No fake percentages
- Real-time state machine status
- Phase-based progress tracking

#### 9.4 Findings Screen Redesign

**New Findings Display**:
- Columns: ID, Severity, Confidence, Title, Asset, Service, CVE, CWE, Scanner Sources, Validation, Status, Risk
- Detailed finding view:
  - Description
  - Affected asset
  - Evidence (with access)
  - Source observations
  - CVE details (CVSS, EPSS, KEV)
  - Remediation steps
  - Validation procedures
  - Timeline and history

#### 9.5 CVE Intelligence Screen

**New Intelligence Display**:
- Database version and last update
- Last successful import timestamp
- Record counts (CVE, CPE, KEV, EPSS)
- Source health indicators
- Index health status
- Offline capability status
- Actions:
  - Check for updates
  - Download update
  - Verify integrity
  - Import to staging
  - Promote to production
  - Rollback
  - Rebuild indexes
  - Verify database
  - Export intelligence package

#### 9.6 Implementation Tasks

**UI Framework**:
1. Redesign navigation structure
2. Create new screen templates
3. Implement data model bindings
4. Add real-time update mechanisms
5. Create responsive layouts

**Dashboard**:
1. Implement new dashboard metrics
2. Add risk trend visualization
3. Create active scan monitoring
4. Add intelligence health indicators
5. Build system status overview

**Screens**:
1. Create engagements screen
2. Implement assets screen
3. Build services screen
4. Create technologies screen
5. Implement findings screen with detail view
6. Build evidence browser
7. Create risk screen
8. Implement remediation tracking
9. Build CVE intelligence screen
10. Create audit log viewer

**Integration**:
1. Wire UI to new backend APIs
2. Implement real-time data updates
3. Add error handling and retry
4. Create loading states
5. Build offline mode indicators

**Testing**:
1. Add UI component tests
2. Test screen navigation
3. Add data binding tests
4. Test real-time updates
5. Create UX testing

**Documentation**:
1. Create `docs/ui/architecture.md`
2. Document UI components
3. Create user guide updates
4. Document screen workflows
5. Create UI development guide

**Acceptance Criteria**:
- Navigation reflects new data model
- Dashboard shows accurate metrics
- Scan progress is real and accurate
- Findings display shows complete information
- CVE intelligence screen is functional
- UI is responsive and performant
- Real-time updates work correctly
- Error handling is user-friendly

**Git Commit**:
```
git add .
git commit -m "phase-9: refactor ui for new architecture"
```

### PHASE 10 — REPORT GENERATION & EXPORTER

**Objective**: Transform report generator to consume finalized security model.

#### 10.1 Report Pipeline Redesign

**New Report Pipeline**:

```
Engagement
+ Scope
+ Assets
+ Services
+ Technologies
+ Findings
+ Evidence
+ Risk
+ Validation
+ Remediation
+ Intel Provenance
    ↓
Report Generation
```

**Report Types**:
- Executive report
- Technical VAPT report
- Finding evidence report
- Asset inventory
- CVE exposure report
- Remediation report
- Retest report
- Machine-readable JSON
- CSV exports
- SBOM generation

#### 10.2 Report Authenticity Enhancement

**Enhanced Content Hash**:

```python
REPORT_HASH_INPUTS = {
    "finding_ids": "list[UUID]",
    "evidence_hashes": "list[SHA256]",
    "intelligence_version": "str",
    "scanner_versions": "dict",
    "report_generator_version": "str",
    "engagement_id": "UUID",
    "timestamp": "ISO8601",
    "canonical_data": "dict (canonicalized report data)"
}
```

**Hash Computation**:
- Canonicalize all report data
- Include finding IDs and evidence hashes
- Include intelligence version
- Include scanner versions
- Include report generator version
- Include engagement ID and timestamp
- Optional detached signature support

#### 10.3 Report Content Enhancement

**Enhanced Report Sections**:
1. **Document Control & Cover Page**
   - Enhanced with engagement metadata
   - Scope and authorization details
   - Team information

2. **Executive Summary**
   - Risk score and trend
   - Critical findings summary
   - Asset exposure overview
   - Intelligence freshness

3. **Engagement Scope & Methodology**
   - Detailed scope rules
   - Authorization records
   - Scan methodology
   - Tools and versions used

4. **Findings Summary Matrix**
   - Enhanced with evidence links
   - CVE intelligence integration
   - Risk scoring details
   - Scanner source attribution

5. **Deep-Dive Technical Findings**
   - Complete evidence attachments
   - Source observation details
   - CVE technical details
   - Validation procedures
   - Remediation steps

6. **Appendices**
   - Complete asset inventory
   - Service inventory
   - Technology inventory
   - CVE exposure analysis
   - Intelligence provenance
   - Tool version manifest
   - Attestation

#### 10.4 Implementation Tasks

**Report Pipeline**:
1. Redesign report data consumption
2. Integrate with new security model
3. Add evidence inclusion
4. Implement intelligence provenance
5. Create scope documentation

**Report Types**:
1. Enhance executive report
2. Update technical VAPT report
3. Create finding evidence report
4. Build asset inventory report
5. Implement CVE exposure report
6. Create remediation report
7. Build retest report
8. Add machine-readable exports
9. Implement SBOM generation

**Authenticity**:
1. Implement enhanced hash computation
2. Add canonicalization
3. Include all required inputs
4. Create detached signature support
5. Build hash verification

**Integration**:
1. Wire report generator to new APIs
2. Add evidence retrieval
3. Include intelligence data
4. Implement scope documentation
5. Create version tracking

**Testing**:
1. Add report generation tests
2. Test hash computation
3. Verify evidence inclusion
4. Test all report types
5. Create performance tests

**Documentation**:
1. Create `docs/architecture/report_generator.md`
2. Document report pipeline
3. Create report template guide
4. Document authenticity mechanism
5. Create report customization guide

**Acceptance Criteria**:
- Reports consume new security model
- Evidence is properly included
- Intelligence provenance is documented
- Hash computation is comprehensive
- All report types work correctly
- Reports are accurate and complete
- Performance is acceptable
- Authenticity can be verified

**Git Commit**:
```
git add .
git commit -m "phase-10: enhance report generation and exporter"
```

## Complete Implementation Summary

This comprehensive plan covers all 10 phases of the SMP enterprise-grade rebuild:

1. **Phase 1**: Foundation with separated databases, enhanced security, and data contracts
2. **Phase 2**: Engagement model, scope engine, and intelligent scan planning
3. **Phase 3**: Scanner execution framework with verification and observation-based output
4. **Phase 4**: Vulnerability intelligence system with offline capability
5. **Phase 5**: Observation model and evidence engine
6. **Phase 6**: Scan orchestrator redesign with component separation
7. **Phase 7**: Evidence-preserving deduplication and finding model
8. **Phase 8**: Nmap integration as primary service discovery source
9. **Phase 9**: UI refactoring for new architecture
10. **Phase 10**: Enhanced report generation and exporter

The plan follows the user's constraints:
- Breaking changes acceptable (V10.0)
- Framework before scanner migration
- File storage with metadata for evidence
- Complete all 10 phases sequentially
- UI refactoring after backend stable

Each phase includes detailed implementation tasks, acceptance criteria, testing requirements, and documentation guidelines to ensure a successful enterprise-grade transformation.
