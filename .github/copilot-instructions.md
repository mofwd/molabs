# M&O Innovation Lab - AI Coding Agent Instructions

**Version**: 1.0  
**Last Updated**: 2025-10-14  
**Target**: GitHub Copilot, Cursor, and other AI coding assistants

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Core Principles](#core-principles)
3. [Technology Standards](#technology-standards)
4. [Testing Requirements](#testing-requirements)
5. [Security Practices](#security-practices)
6. [Observability & SRE](#observability--sre)
7. [Infrastructure Patterns](#infrastructure-patterns)
8. [n8n Integration Patterns](#n8n-integration-patterns)
9. [Git Workflow & Issue Integration](#git-workflow--issue-integration)
10. [Documentation Standards](#documentation-standards)
11. [AI Pair Programming Guidelines](#ai-pair-programming-guidelines)

---

## Project Overview

**M&O Innovation Lab Infrastructure Project** is M&O Companies' centralized planning hub for its M&O Innovation Lab (aka Molabs). It provides a comprehensive overview of our lab's modernization and automation efforts. This repository serves as a reference for product managers, executives, and devops teams to understand our various projects' purpose, value proposition, key assumptions, and key hypotheses.

### Repository as Product Backlog

This GitHub repository's issues serve as our **product backlog**, with different issue types mapping directly to traditional backlog item types:

- `epic` label → Epic (parent of multiple stories)
- `story` label → Story (user-facing functionality)
- `chore` label → Task (technical work, infrastructure)
- `bug` label → Bug (something broken)
- `enhancement` label → Feature Request (new functionality)

All Stories, Feature Requests, and Bugs receive a `needs-triage` label by default, removed after prioritization.

### Current Status

Several projects are in the planning phase, with some initial implementations underway. The focus is on creating a **secure, scalable, and automated infrastructure** that can support various applications and services, while also piloting initial integrations that will prove out the architecture and automation patterns.

**Key Initiatives:**
- 🚀 Establishing TDD culture and automated testing
- 🚀 n8n workflow version control
- 🚀 ProxMox automation (Terraform/Ansible)
- ✅ Grafana observability stack operational
- ✅ CloudFlare Zero Trust security model deployed
- ✅ Vaultwarden secret management established

### Strategic Context

**Vision:** See `docs/vision.md` for long-term product vision and objectives

**Architecture:** See `docs/architecture.md` for infrastructure topology, network design, and CloudFlare tunnel configuration

**Future Ideas:** See `docs/ideas.md` for planned expansions and integration concepts

### Audience for This Document

This document is designed to train **AI coding assistants** (GitHub Copilot, Cursor, etc.) to generate code and suggestions that align with Molabs Infrastructure standards. The document also serves as a **quick reference** for:

- DevOps engineers implementing infrastructure
- Product managers understanding technical constraints
- Developers contributing to automation projects
- AI assistants providing contextual code generation

---

## Core Principles

### Product & Development Philosophy

**1. Outcome Over Output**  
Code solves real user problems; features exist to test hypotheses. When generating code, always ask: "What user problem does this solve?" and "What hypothesis are we testing?"

**2. Hypothesis-Driven Development**  
State assumptions explicitly; every significant change tests one. Include hypothesis statements in commit messages and PR descriptions for non-trivial changes.

**3. Test-First Discipline**  
Write the failing test first, then make it pass. We're establishing TDD culture - help reinforce this by suggesting tests before implementations.

**4. Fail Fast, Learn Faster**  
Small, reversible commits over large, risky changes. Prefer incremental refactoring with working tests at each step.

**5. User Empathy Over Expertise**  
Validate with users; we don't know better than they do. When suggesting features, frame them as testable hypotheses, not assumptions.

**6. Continuous Integration**  
Main branch is always deployable. Never suggest code that breaks existing tests or deployability.

### Infrastructure & Architecture

**7. Infrastructure as Code**  
All automation versioned in Git (bash, Python, Dockerfiles, n8n workflows). Never suggest manual configuration steps - always provide scriptable, version-controllable solutions.

**8. n8n-First Orchestration**  
Complex workflows and integrations belong in n8n, not scattered scripts. When suggesting multi-step automations, consider whether n8n is the right tool.

**9. Data Ownership via APIs**  
We own or control access to our data; integrate, don't vendor-lock. When interfacing with external services (Airtable, Softr), always prioritize API access patterns that preserve data portability.

**10. Observability by Default**  
Every host/container must ship logs and metrics to Grafana stack (Prometheus/Loki). All code should include appropriate logging and metrics exposition.

### Security & Operations

**11. Secrets in Vaultwarden Only**  
All credentials, tokens, and SSH keys stored in `vw.molabs.tools`; never hardcoded. Never suggest inline secrets - always reference environment variables or Vaultwarden retrieval.

**12. Least Privilege Always**  
Grant minimum necessary access; audit regularly. When creating users, services, or permissions, default to restrictive and expand only as needed.

**13. Ruthless Simplicity**  
Simplest working solution wins; complexity is technical debt. Avoid over-engineering. Suggest straightforward solutions before complex ones.

---

## Technology Standards

### Context Documents
Reference these files for detailed infrastructure context:
- `docs/architecture.md` - Network topology, ProxMox hosts, CloudFlare tunnels, VM details
- `docs/ideas.md` - Expansion and integration concepts
- `docs/vision.md` - Long-term product vision

### ProxMox & Virtualization

**Environment:**
- Two ProxMox hosts: `pm1` (Hetzner cloud) and `pm2` (Mokena LAN)
- VM Networks: `10.50.0.0/24` (pm1), `10.51.0.0/24` (pm2)
- Ubuntu 24.04 LTS is standard for Linux VMs
- Windows Server 2022+ and Windows 11 for Windows VMs

**Key Principles:**
- All external access via CloudFlare Zero Trust tunnels (see `docs/architecture.md`)
- ProxMox automation is a priority initiative (not yet implemented)
- VM creation currently manual; automation coming soon

### Bash Scripting Standards

**Every script must start with:**
```bash
#!/usr/bin/env bash
set -e          # Exit on error
set -u          # Exit on undefined variable
set -o pipefail # Exit on pipe failure

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
```

**Core Principles:**
- **Idempotency**: Check existence before creating (files, users, mounts, etc.)
- **Logging**: Use color-coded functions from `scripts/common.sh`, never raw `echo`
- **Validation**: Preflight checks validate, execution scripts assume valid input
- **Error Handling**: Always handle errors gracefully with informative messages

**Example Pattern:**
```bash
# Good: Idempotent directory creation
if [[ ! -d "/opt/myapp" ]]; then
    log_info "Creating /opt/myapp directory"
    mkdir -p /opt/myapp
fi

# Bad: Non-idempotent, no logging
mkdir /opt/myapp
```

**Logging Functions (from `scripts/common.sh`):**
```bash
log_info "message"    # Blue text for informational messages
log_success "message" # Green text for success messages
log_warning "message" # Yellow text for warnings
log_error "message"   # Red text for errors
```

### Python Standards

**Version Requirements:**
- Python 3.11+ required (3.11 specifically needed for Pervasive ODBC integration on Windows)
- Use type hints where practical
- Follow PEP 8 generally, but not strictly enforced yet

**Virtual Environments:**
- Use any tool (venv, poetry, pipenv) - project-specific choice
- Always include requirements.txt or equivalent
- Document setup in README.md

**Example Structure:**
```python
#!/usr/bin/env python3
"""
Brief module description.

This module does X to solve Y problem.
"""

import sys
from typing import Optional

def main() -> int:
    """Main entry point."""
    try:
        # Implementation
        return 0
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        return 1

if __name__ == "__main__":
    sys.exit(main())
```

### PowerShell Standards

**Version & Environment:**
- PowerShell 7+ preferred (cross-platform, modern)
- Windows Server 2022+ and Windows 11 workstations
- PowerShell 5.1 acceptable for Windows-only scripts

**Error Handling:**
```powershell
#Requires -Version 7.0
$ErrorActionPreference = 'Stop'

try {
    # Script logic
} catch {
    Write-Error "Failed: $_"
    exit 1
}
```

**Best Practices:**
- Use approved verbs (Get-, Set-, New-, Remove-, etc.)
- Include comment-based help
- Use `[CmdletBinding()]` for advanced functions
- Follow Microsoft's PowerShell style guide

### Docker & Container Standards

**Restart Policy:**
```yaml
restart: unless-stopped  # Automatic recovery without interfering with manual stops
```

**Data Persistence:**
- Use project-root-relative volume mounts: `./data/service-name:/container/path`
- Never use unnamed volumes in production
- Document volume purposes in docker-compose.yml comments

**Logging Integration (Loki):**
```yaml
services:
  myservice:
    image: myapp:latest
    logging:
      driver: loki
      options:
        loki-url: "http://10.50.0.20:3100/loki/api/v1/push"  # pm1 VMs
        # loki-url: "http://10.51.0.4:3100/loki/api/v1/push"  # pm2 VMs
        loki-retries: 2
        loki-batch-size: 400
        loki-external-labels: "service=myservice,host={{.Node.Hostname}},environment=production"
        mode: non-blocking
        max-buffer-size: 4m
        keep-file: "true"
```

**Loki Endpoint Selection:**
- pm1 VMs: `http://10.50.0.20:3100/loki/api/v1/push`
- pm2 VMs: `http://10.51.0.4:3100/loki/api/v1/push`

**Network Configuration:**
- Use explicit networks, avoid default bridge
- Document port mappings clearly
- Consider CloudFlare tunnel exposure requirements

**Image Standards:**
- Publish to GitHub Container Registry: `ghcr.io/mo-companies/[service-name]:[version]`
- Use semantic versioning tags
- Include `latest` tag for convenience
- Multi-stage builds for smaller production images

### Environment Configuration

**Pattern:**
- All configuration in `.env` file (gitignored)
- Provide `.env.example` template with all required variables
- Document each variable's purpose in `.env.example`

**Example `.env.example`:**
```bash
# Service Configuration
SERVICE_NAME=myservice
SERVICE_PORT=8080

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=myapp_user
DB_PASSWORD=changeme  # Retrieve from vw.molabs.tools

# External Services
API_TOKEN=changeme    # Retrieve from vw.molabs.tools
```

---

## Testing Requirements

**Status:** 🚀 **Priority Initiative** - Establishing TDD culture

### Progressive Coverage Goals

**Phase 1 (Current):**
- All new functions/methods should have at least one test
- Critical paths must have tests before deployment
- Bug fixes must include regression tests

**Phase 2 (6 months):**
- 60% code coverage for new projects
- 40% coverage for legacy projects

**Phase 3 (12 months):**
- 80% code coverage for all projects
- Integration test suites for all services

### Framework Selection

**Bash:**
- Recommend: bats-core for test framework
- Place tests in `tests/` directory
- Name pattern: `test_*.bats`

**Python:**
- Recommend: pytest
- Place tests in `tests/` directory
- Name pattern: `test_*.py`
- Use fixtures for common setup

**Docker:**
- Recommend: container-structure-test
- Test container structure, files, commands, metadata
- Place tests in `tests/container/`

### Test Structure Principles

**1. Arrange-Act-Assert Pattern:**
```python
def test_user_creation():
    # Arrange
    username = "testuser"
    
    # Act
    user = create_user(username)
    
    # Assert
    assert user.username == username
    assert user.is_active == True
```

**2. Test Naming:**
```python
# Good: Describes what and why
def test_email_validation_rejects_invalid_format():
    pass

# Bad: Vague
def test_email():
    pass
```

**3. Test Independence:**
- Each test should run independently
- No shared state between tests
- Use fixtures/setup for common initialization

### When to Write Tests

**Always:**
- Before implementing new features (TDD)
- When fixing bugs (regression prevention)
- For critical business logic
- For public APIs/interfaces

**Consider:**
- Integration tests for service interactions
- End-to-end tests for critical user flows
- Performance tests for high-load components

---

## Security Practices

**Status:** ✅ **Established**

### Vaultwarden - Mandatory Secret Management

**All credentials must be stored in Vaultwarden:**
- Instance: `vw.molabs.tools`
- Access via Bitwarden clients only
- SSH keys managed via ssh-agent function

**Never:**
- Hardcode secrets in source code
- Commit secrets to Git
- Share secrets via chat/email
- Store secrets in plaintext files

**Always:**
- Reference secrets via environment variables
- Document in `.env.example` that values come from Vaultwarden
- Use descriptive comments: `# Retrieve from vw.molabs.tools`

### SSH Key Management

**Pattern:**
```bash
# Developer workflow
# 1. Store SSH key in Vaultwarden
# 2. Use ssh-agent integration
# 3. Keys loaded automatically from Vaultwarden

# Never store keys in:
~/.ssh/id_rsa  # Bad
./deploy_key   # Bad
```

### API Token Handling

**In Code:**
```python
# Good
api_token = os.environ.get('API_TOKEN')
if not api_token:
    raise ValueError("API_TOKEN not set - retrieve from vw.molabs.tools")

# Bad
api_token = "sk_live_abc123..."  # Never hardcode
```

**In Docker Compose:**
```yaml
services:
  myservice:
    environment:
      - API_TOKEN=${API_TOKEN}  # Loaded from .env
```

### Network Security

**CloudFlare Zero Trust:**
- All external access via CloudFlare tunnels
- No direct internet exposure
- See `docs/architecture.md` for tunnel configuration

**Firewall Rules:**
- Default deny posture
- Explicit allow rules only
- Document all port openings

---

## Observability & SRE

**Status:** ✅ **Grafana Stack Operational**

### Grafana Observability Stack

**Components:**
- **Grafana**: Visualization and dashboards
- **Prometheus**: Metrics collection and storage
- **Loki**: Log aggregation
- **Node Exporter**: Host metrics (Linux)
- **Windows Exporter**: Host metrics (Windows)
- **Promtail**: Log shipping

**Production Instances:**
- pm1: Grafana stack at `10.50.0.20`
- pm2: Grafana stack at `10.51.0.4`

### Prometheus Metrics

**All services should expose metrics:**
```python
from prometheus_client import Counter, Histogram, start_http_server

# Define metrics
requests_total = Counter('http_requests_total', 'Total HTTP requests')
request_duration = Histogram('http_request_duration_seconds', 'HTTP request duration')

# Use in code
requests_total.inc()
with request_duration.time():
    # Handle request
    pass

# Expose metrics endpoint
start_http_server(9090)  # Metrics at :9090/metrics
```

**Metric Naming:**
- Format: `[namespace]_[subsystem]_[name]_[unit]`
- Example: `myapp_http_requests_total`
- Use base units (seconds, bytes, not milliseconds/megabytes)

### Loki Structured Logging

**Docker Logging (see Container Standards section):**
```yaml
logging:
  driver: loki
  options:
    loki-external-labels: "service=myservice,environment=production"
```

**Application Logging:**
```python
import logging
import json

# Structured logging
log = logging.getLogger(__name__)
log.info(json.dumps({
    "event": "user_login",
    "user_id": user.id,
    "ip": request.remote_addr,
    "timestamp": datetime.utcnow().isoformat()
}))
```

**Log Levels:**
- ERROR: Something failed that requires attention
- WARNING: Something unexpected but handled
- INFO: Normal operation events
- DEBUG: Detailed diagnostic information

### Monitoring Best Practices

**Four Golden Signals:**
1. **Latency**: How long requests take
2. **Traffic**: How many requests
3. **Errors**: Rate of failed requests
4. **Saturation**: How "full" the service is

**Implement for every service:**
```python
# Latency
request_duration_histogram.observe(duration)

# Traffic
requests_total_counter.inc()

# Errors
errors_total_counter.inc()

# Saturation
current_connections_gauge.set(active_connections)
```

---

## Infrastructure Patterns

**Status:** 🚀 **Automation Initiatives In Progress**

### ProxMox Virtualization

**Current State:**
- Manual VM provisioning
- Two hosts: pm1 (cloud), pm2 (Mokena LAN)
- ZFS storage for snapshots and expansion

**Network Architecture:**
- pm1 VMs: `10.50.0.0/24` network
- pm2 VMs: `10.51.0.0/24` network
- ProxMox hosts serve as gateways (`.1` address)
- Internal DNS resolution at gateway

**VM Standards:**
- Ubuntu 24.04 LTS for Linux VMs
- Windows Server 2022+ for Windows VMs
- Proper DNS search domain configuration
- Node exporter/Windows exporter for metrics
- Promtail for log shipping

### CloudFlare Tunnel Architecture

**See `docs/architecture.md` for complete tunnel configuration**

**Key Principles:**
- All external access via CloudFlare Zero Trust
- No direct internet exposure
- Service-specific public hostnames
- TLS configuration for self-signed certs

**When exposing new services:**
1. Deploy service on internal network
2. Add public hostname to appropriate tunnel
3. Configure TLS settings if needed
4. Test access through CloudFlare

### Configuration Management

**Current Approach:**
- Environment variables via `.env` files
- Manual configuration with documentation
- Scripts for repeatable setup steps

**Future Direction:**
- Ansible for configuration management
- Terraform for infrastructure provisioning
- Idempotent automation scripts

---

## n8n Integration Patterns

**Status:** 🚀 **Version Control Initiative**

### When to Use n8n

**Use n8n for:**
- Complex multi-step workflows
- Integration between multiple services
- Scheduled tasks with multiple dependencies
- Data transformation pipelines
- Event-driven automation

**Use scripts for:**
- Simple one-off tasks
- System administration
- Deployment automation
- CI/CD processes

### Workflow Organization

**Directory Structure:**
```
n8n-workflows/
├── production/
│   ├── customer_data_sync_w123.json
│   ├── invoice_processing_w456.json
│   └── backup_automation_w789.json
└── development/
    ├── test_workflow_w999.json
    └── experimental_workflow_w888.json
```

**Naming Convention:**
- Format: `[lowercase_workflow_name]_[workflowId].json`
- Examples:
  - `customer_data_sync_w123.json`
  - `invoice_processing_w456.json`
  - `email_notification_service_w789.json`

### Version Control Strategy

**Backup Current:**
- Manual backup of n8n server
- Backups stored securely

**Target State:**
- Bulk export workflows to Git
- Automated export on workflow changes
- PR review for workflow modifications

### n8n Best Practices

**Workflow Design:**
- Use descriptive node names
- Add notes for complex logic
- Handle errors explicitly
- Include retry logic for external APIs
- Log important state changes

**Data Handling:**
- Validate input data
- Transform data at clear boundaries
- Use variables for repeated values
- Document expected data structures

---

## Git Workflow & Issue Integration

### Branch Strategy

**Branch Types:**
```bash
# Issue-based branches
issue-123-add-user-authentication
issue-456-fix-database-connection

# Feature branches
feature-payment-integration
feature-pdf-generation

# Bug fixes
fix-email-validation
fix-memory-leak
```

**Principles:**
- Always reference issue number when applicable
- Use descriptive, kebab-case names
- No personal branches (no `rob/feature` or `dev/task`)
- Short-lived branches (merge within days, not weeks)

### Commit Messages

**Format:**
```bash
git ac "[short message] (fixes #[issue-number])" -m "[detailed-message]"
```

**Examples:**
```bash
# Good
git ac "Add email relay configuration (fixes #40)" -m "Implemented msmtp-based email relay with hostname prepending and per-user configuration"

git ac "Fix database connection timeout (fixes #127)" -m "Increased connection pool size and added retry logic"

# Also acceptable
git commit -m "Refactor user authentication module

- Split auth logic into separate functions
- Add unit tests for token validation
- Improve error messages
- Related to #89"
```

**Principles:**
- First line: imperative mood, 50 chars or less
- Reference issue with `fixes #123` or `relates to #123`
- Body: Explain why, not what (code shows what)
- Atomic commits (one logical change per commit)

### GitHub Issue Management

**Issue Types (via labels):**
- `epic` → Epic (parent of multiple stories)
- `story` → Story (user-facing functionality)
- `chore` → Task (technical work, no user impact)
- `bug` → Bug (something broken)
- `enhancement` → Feature Request (new functionality)

**Auto-labels:**
- Stories, Feature Requests, and Bugs get `needs-triage` by default
- Remove `needs-triage` after prioritization

**Shell Aliases (in developer `~/.zshrc`):**
```bash
bug "Description"      # Creates: 🐛 Description (labeled: bug, needs-triage)
feature "Description"  # Creates: ✨ Description (labeled: enhancement, needs-triage)
doc "Description"      # Creates: 📝 Description (labeled: documentation)
question "Question"    # Creates: ❓ Question (labeled: question)
todo "Task"           # Creates: Task (no label)
```

**Listing Open Issues:**
```bash
gh issue list --state open --limit 50 --json number,title,labels,state,createdAt | jq -r '.[] | "#\(.number) - \(.title) [\(.labels[].name // "no-label" | @text)]"'
```

### Pull Request Guidelines

**Title Format:**
```
[Type] Short description (fixes #123)

Examples:
[Feature] Add user authentication (fixes #45)
[Bug] Fix memory leak in data processor (fixes #78)
[Chore] Update dependencies
```

**PR Description Template:**
```markdown
## Summary
Brief description of changes

## Related Issues
Fixes #123
Relates to #456

## Changes Made
- Added X feature
- Fixed Y bug
- Refactored Z component

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Deployment Notes
Any special deployment considerations
```

---

## Documentation Standards

### Daily Progress Logs

**Purpose:** Track development decisions and progress for complex projects

**Location:** `docs/progress/YYYY-MM-DD.md`

**When to Update:**
- Throughout the day as work progresses
- Configuration changes and their rationale
- Architecture decisions and tradeoffs
- Integration challenges and solutions
- Testing results and deployment notes

**Template:**
```markdown
# Progress Log - 2025-10-14

## Summary
One-line summary of the day's focus

## Activity Log

### [HH:MM] Activity Title
- What was done
- Why it was done this way
- Any decisions made
- Next steps

### [HH:MM] Another Activity
- Details
- Outcomes
- Learnings
```

**Guidelines:**
- Append to existing day's log (don't create multiple files per day)
- Use timestamps for significant activities
- Include rationale for decisions
- Note blockers and how they were resolved

### README.md Guidelines

**Purpose:** High-level overview for Molabs Infrastructure and DevOps Team

**Structure:**
```markdown
# Project Name

Brief description

## Prerequisites
- Required systems
- Required access
- Required tools

## Quick Start
1. Step-by-step deployment
2. With commands
3. That actually work

## Configuration
Link to detailed docs

## Architecture
Link to docs/architecture.md

## Troubleshooting
Common issues and solutions

## Links
- Detailed documentation
- Related projects
- External resources
```

**Principles:**
- Keep README.md synchronized with current deployment workflow
- Link to detailed docs rather than duplicating
- Target audience: team members who need to deploy/maintain
- Update README.md whenever deployment process changes

### Code Documentation

**When to Document:**
- Public APIs and interfaces
- Complex algorithms
- Non-obvious decisions
- Configuration options
- Setup/deployment procedures

**When Not to Document:**
- Self-explanatory code
- Standard patterns
- Obvious variable names

**Style:**
```python
def process_invoice(invoice_data: dict) -> Invoice:
    """
    Process raw invoice data into Invoice object.
    
    This function handles multiple invoice formats from different
    vendors and normalizes them into our standard Invoice model.
    Special handling for legacy Pervasive database format.
    
    Args:
        invoice_data: Raw invoice dict with vendor-specific structure
        
    Returns:
        Invoice: Validated and normalized invoice object
        
    Raises:
        ValidationError: If invoice data is malformed
        VendorError: If vendor-specific parsing fails
    """
```

### Architecture Documentation

**Keep `docs/architecture.md` updated with:**
- Network topology changes
- New VM deployments
- Service additions
- Tunnel configuration changes
- Infrastructure decisions

---

## AI Pair Programming Guidelines

### How to Use This Document

**This document trains you (the AI assistant) to:**
1. Understand Molabs Infrastructure project context
2. Generate code that follows our standards
3. Suggest solutions aligned with our principles
4. Reference appropriate documentation

### Effective Prompts

**Good Prompts:**
```
"Create a bash script to deploy a new service with:
- Idempotent directory creation
- Logging using common.sh functions
- Error handling with set -e
- Docker Compose with Loki logging"

"Write a Python function to process CSV data with:
- Type hints
- Error handling
- Unit tests using pytest
- Logging to stdout"

"Generate a docker-compose.yml for a web service with:
- Loki logging to pm1
- Environment variables from .env
- Restart policy: unless-stopped
- Volume mount to ./data/"
```

**Poor Prompts:**
```
"Make a script"  # Too vague
"Fix this"       # No context
"Add logging"    # Which logging system? What standards?
```

### Code Generation Expectations

**When generating code, AI should:**
1. Follow technology standards from this document
2. Include appropriate error handling
3. Add logging/observability instrumentation
4. Never include hardcoded secrets
5. Include basic tests when appropriate
6. Reference environment variables for configuration
7. Use idempotent patterns
8. Add brief comments for non-obvious logic

**When suggesting architecture:**
1. Consider CloudFlare tunnel exposure
2. Reference Loki endpoints for appropriate ProxMox host
3. Suggest n8n for complex workflows
4. Prioritize simple solutions
5. Consider observability from the start

### Iterative Development Pattern

**Recommended flow:**
1. User describes problem/feature
2. AI suggests test-first approach
3. AI generates failing test
4. AI generates implementation
5. AI suggests how to verify
6. User runs/validates
7. Iterate

### When to Reference Documentation

**AI should prompt user to check:**
- `docs/architecture.md` for infrastructure details
- `docs/ideas.md` for planned features
- `docs/vision.md` for product direction
- `.env.example` for configuration options
- `docs/progress/` for recent decisions

### Red Flags - Never Suggest

❌ Hardcoded secrets or credentials  
❌ Code without error handling  
❌ Services without logging  
❌ Docker containers without restart policies  
❌ Bypassing Vaultwarden for secrets  
❌ Direct internet exposure (bypass CloudFlare)  
❌ Complex solutions when simple ones suffice  
❌ Breaking changes without tests  

### Green Lights - Always Suggest

✅ Environment variables for configuration  
✅ Tests before implementation (TDD)  
✅ Logging and metrics instrumentation  
✅ Idempotent scripts and automation  
✅ Simple, readable solutions  
✅ Documentation for non-obvious code  
✅ References to Vaultwarden for secrets  
✅ CloudFlare tunnel for external access  

---

## Quick Reference Card

### Essential Commands

```bash
# Git & Issues
git ac "message (fixes #123)" -m "details"
bug "Description"
feature "Description"
gh issue list --state open --limit 50

# Development
pytest tests/
docker-compose up -d
docker-compose logs -f

# Infrastructure
ssh user@host  # Keys from vw.molabs.tools via ssh-agent
```

### Key Files

```
.env                        # Local config (gitignored)
.env.example                # Config template
docs/architecture.md        # Infrastructure topology
docs/progress/YYYY-MM-DD.md # Daily development log
scripts/common.sh           # Bash logging functions
```

### Critical URLs

- Vaultwarden: `vw.molabs.tools`
- pm1 Loki: `http://10.50.0.20:3100/loki/api/v1/push`
- pm2 Loki: `http://10.51.0.4:3100/loki/api/v1/push`
- Architecture Docs: `docs/architecture.md`

---

## Version History

- **1.0** (2025-10-14): Initial release
  - Core principles established
  - Technology standards documented
  - Testing framework outlined
  - Security practices codified
  - Observability patterns defined

---

**Questions or suggestions?** Update this document and submit a PR with the `documentation` label.