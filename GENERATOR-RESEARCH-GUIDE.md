# Generator Research Guide

This document describes the step-by-step process for investigating any OpenAPI Generator to assess its capabilities against GOAL_MAX.md requirements.

**Reference:** This guide was developed during investigation of `php-laravel` (GENDE-010) and `php-symfony` (GENDE-003).

---

## Key Principle: Demo Project Per Option

**Each generator option should have its own demo project.** This is critical because:

1. **Shows real integration patterns** - How to actually use the generated code
2. **Reveals integration limitations** - What doesn't work becomes obvious
3. **Validates generated code** - Proves it compiles and runs
4. **Reference for users** - Copy-paste starting point

**Demo Project Naming Convention:**
```
projects/{framework}-api--{generator}--{variant}/

Examples:
- projects/laravel-api--php-laravel--default/      # OOTB php-laravel
- projects/laravel-api--php-laravel--custom/       # Custom templates
- projects/laravel-api--laravel-max--default/      # Custom generator
- projects/symfony-api--php-symfony--default/      # OOTB php-symfony
```

---

## Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Generator Research Pipeline                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PHASE 1-4: OOTB Investigation                                          │
│  ═══════════════════════════════                                        │
│                                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐         │
│  │ Extract  │───▶│ Generate │───▶│ Demo     │───▶│ Analyze  │         │
│  │ Templates│    │ Sample   │    │ Project  │    │ & Score  │         │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘         │
│       │               │               │               │                 │
│       ▼               ▼               ▼               ▼                 │
│   templates/      generated/      projects/      GENERATOR-             │
│   {gen}-default   {gen}/          {fw}--{gen}    ANALYSIS.md           │
│                                   --default/                            │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PHASE 5-7: Custom Templates (if proceeding)                            │
│  ═══════════════════════════════════════════                            │
│                                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                          │
│  │ Custom   │───▶│ Demo     │───▶│ Integr.  │                          │
│  │ Templates│    │ Project  │    │ Tests    │                          │
│  └──────────┘    └──────────┘    └──────────┘                          │
│       │               │               │                                 │
│       ▼               ▼               ▼                                 │
│   templates/      projects/       tests/                                │
│   {gen}/          {fw}--{gen}                                           │
│                   --custom/                                             │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PHASE 8-9: Custom Generator (if needed)                                │
│  ═══════════════════════════════════════                                │
│                                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                          │
│  │ Custom   │───▶│ Demo     │───▶│ Full     │                          │
│  │ Generator│    │ Project  │    │ Tests    │                          │
│  └──────────┘    └──────────┘    └──────────┘                          │
│       │               │               │                                 │
│       ▼               ▼               ▼                                 │
│   generators/     projects/       tests/                                │
│   {fw}-max/       {fw}--{fw}-max                                        │
│                   --default/                                            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Artifact Matrix

| Phase | Templates | Generated | Demo Project | Tests | Analysis |
|-------|-----------|-----------|--------------|-------|----------|
| OOTB | `{gen}-default/` | `{gen}/` | `{fw}--{gen}--default/` | Basic | GENERATOR-ANALYSIS.md |
| Custom Templates | `{gen}/` | `{gen}/` | `{fw}--{gen}--custom/` | Integration | Updated analysis |
| Custom Generator | N/A (Java) | `{fw}-max/` | `{fw}--{fw}-max--default/` | Full | Full comparison |

---

## Phase 1: Extract Default Templates

### Purpose
Get the original templates to understand what the generator produces out-of-the-box.

### Commands

```bash
# Create target directory
mkdir -p openapi-generator-server-templates/openapi-generator-server-{generator}-default

# Extract templates
docker run --rm \
  -v "$(pwd)/openapi-generator-server-templates:/local" \
  openapitools/openapi-generator-cli:v7.12.0 \
  author template \
  -g {generator} \
  -o /local/openapi-generator-server-{generator}-default
```

### Examples

| Generator | Command |
|-----------|---------|
| php-laravel | `-g php-laravel` |
| php-symfony | `-g php-symfony` |
| php-slim | `-g php-slim4` |
| php-mezzio | `-g php-mezzio-ph` |
| typescript-express | `-g typescript-express-server` |

### Output Structure

```
openapi-generator-server-templates/
└── openapi-generator-server-{generator}-default/
    ├── *.mustache           # Template files
    ├── .helpers/            # Helper templates (if any)
    └── (other files)
```

### Checklist
- [ ] Templates extracted successfully
- [ ] Template files listed and counted
- [ ] Directory structure documented

---

## Phase 2: Generate Sample Output

### Purpose
Generate actual code to analyze what the generator produces.

### Prerequisites
- OpenAPI specs available in `openapi-generator-specs/`
- Both TicTacToe and PetShop specs recommended

### Commands

```bash
# Create output directory
mkdir -p generated/{generator}

# Generate TicTacToe API
docker run --rm \
  -v "$(pwd):/local" \
  openapitools/openapi-generator-cli:v7.12.0 generate \
  -g {generator} \
  -i /local/openapi-generator-specs/tictactoe/tictactoe.json \
  -o /local/generated/{generator}/tictactoe

# Generate PetShop API
docker run --rm \
  -v "$(pwd):/local" \
  openapitools/openapi-generator-cli:v7.12.0 generate \
  -g {generator} \
  -i /local/openapi-generator-specs/petshop/petshop.json \
  -o /local/generated/{generator}/petshop
```

### Output Structure

```
generated/
└── {generator}/
    ├── tictactoe/
    │   ├── src/ or lib/ or app/
    │   ├── composer.json (PHP) or package.json (Node)
    │   └── ...
    └── petshop/
        └── ...
```

### Checklist
- [ ] TicTacToe generated without errors
- [ ] PetShop generated without errors
- [ ] Generated file structure documented
- [ ] File counts noted (controllers, models, etc.)

---

## Phase 3: Analyze Against GOAL_MAX.md

### Purpose
Score the generator's output against our quality requirements.

### Scoring Template

Use this table structure in your analysis:

| Requirement | Score | Notes |
|-------------|-------|-------|
| **1. Routes File** | ✅/⚠️/❌ % | |
| **2. Controllers** | ✅/⚠️/❌ % | Per-tag or per-operation? |
| **3. Middleware Support** | ✅/⚠️/❌ % | |
| **4. Security Middleware** | ✅/⚠️/❌ % | Interface + Stub + Validator? |
| **5. Validators** | ✅/⚠️/❌ % | Request validation from schema? |
| **6. API Interfaces** | ✅/⚠️/❌ % | Handler contracts? Return types? |
| **7. Response Classes** | ✅/⚠️/❌ % | Per-response DTOs? |
| **8. Response Factories** | ✅/⚠️/❌ % | Type-safe builders? |
| **9. DTOs/Models** | ✅/⚠️/❌ % | Typed properties? |
| **10. Documentation** | ✅/⚠️/❌ % | Generated docs? |

### Scoring Guide

| Score | Meaning |
|-------|---------|
| ✅ 80-100% | Fully meets requirement |
| ⚠️ 40-79% | Partially meets, gaps exist |
| ❌ 0-39% | Does not meet requirement |

### Key Questions to Answer

1. **Controller Granularity**
   - One controller per operation? (ideal)
   - One controller per tag? (common limitation)
   - Single controller for all? (problematic)

2. **Type Safety**
   - PHP 8.1+ typed properties?
   - Union return types?
   - Strict typing declared?

3. **Validation**
   - Auto-generated from OpenAPI schema?
   - Framework-native (FormRequest, Symfony Validator)?
   - Custom implementation?

4. **Response Handling**
   - Per-response code Resources/DTOs?
   - HTTP status code enforcement?
   - Header validation?

5. **Security**
   - Middleware/authenticator support?
   - Interface contracts for auth?
   - Validation mechanism?

### Checklist
- [ ] All 10 requirements scored
- [ ] Overall percentage calculated
- [ ] Key limitations identified
- [ ] Strengths documented

---

## Phase 4: Document Findings (GENERATOR-ANALYSIS.md)

### Purpose
Create permanent documentation of the generator's capabilities.

### File Location

```
openapi-generator-server-templates/
└── openapi-generator-server-{generator}-default/
    └── GENERATOR-ANALYSIS.md
```

### Template Structure

```markdown
# {Generator} Generator Analysis

**Generator:** `{generator-name}`
**Version:** OpenAPI Generator X.Y.Z
**Analysis Date:** YYYY-MM-DD
**Spec Used:** TicTacToe API, PetShop API

## Overview

Brief description of what the generator produces.

## Generated Structure

\`\`\`
output/
├── (directory tree)
\`\`\`

## Scoring Against GOAL_MAX.md

| Requirement | Score | Notes |
|-------------|-------|-------|
| ... | ... | ... |

**Overall Score: XX%**

## Detailed Analysis

### ✅ Strengths
(What the generator does well)

### ❌ Weaknesses
(Fundamental limitations)

### ⚠️ Limitations for GOAL_MAX.md Compliance
(Specific gaps)

## Comparison with {other-generator}

| Aspect | This Generator | Other Generator |
|--------|---------------|-----------------|
| ... | ... | ... |

## Recommendations

### If Customizing Templates
(What can be improved via templates)

### Effort Estimate for GOAL_MAX Compliance

| Task | Effort |
|------|--------|
| Custom templates | Low/Medium/High |
| Custom Java generator | Required/Not Required |

## Conclusion

Summary and recommendation.

## Files Reference

- Templates: `path/to/templates`
- Generated sample: `path/to/generated`
```

### Checklist
- [ ] GENERATOR-ANALYSIS.md created
- [ ] All sections filled out
- [ ] Code examples included
- [ ] Comparison with existing generators

---

## Phase 5: Create Custom Templates (Optional)

### Purpose
Improve upon default templates where possible within template limitations.

### Directory Structure

```
openapi-generator-server-templates/
├── openapi-generator-server-{generator}-default/  # Reference (don't modify)
└── openapi-generator-server-{generator}/          # Custom templates
    ├── README.md                                  # Document improvements
    └── *.mustache                                 # Modified templates
```

### Process

1. Copy default templates to custom location
2. Identify improvement opportunities
3. Modify templates
4. Generate and verify output
5. Document changes in README.md

### Generation with Custom Templates

```bash
docker run --rm \
  -v "$(pwd):/local" \
  openapitools/openapi-generator-cli:v7.12.0 generate \
  -g {generator} \
  -i /local/openapi-generator-specs/tictactoe/tictactoe.json \
  -o /local/generated/{generator}/tictactoe \
  -t /local/openapi-generator-server-templates/openapi-generator-server-{generator}
```

### Checklist
- [ ] Custom templates directory created
- [ ] Templates copied from default
- [ ] Improvements applied
- [ ] README.md documents changes
- [ ] Both specs generate successfully

---

## Phase 6: Create Demo Project

### Purpose
Prove generated code works in a real application.

### Directory Structure

```
projects/
└── {framework}-api--{generator}--default/
    ├── docker-compose.yml
    ├── Dockerfile
    ├── Makefile
    ├── composer.json / package.json
    ├── src/ or app/
    ├── config/
    ├── tests/
    └── README.md
```

### Makefile Commands (Standard)

```makefile
setup:          # Install dependencies
start:          # Start development server
stop:           # Stop services
test:           # Run tests
generate:       # Regenerate API code
logs:           # View logs
```

### Integration Pattern

1. Install generated code as dependency or link
2. Register routes/controllers
3. Configure DI container
4. Implement sample handlers
5. Test endpoints respond

### Checklist
- [ ] Project directory created
- [ ] Docker environment configured
- [ ] Both APIs integrated
- [ ] Sample handlers implemented
- [ ] `make setup && make start` works
- [ ] At least one endpoint responds

---

## Phase 7: Add Integration Tests

### Purpose
Validate generated code works correctly with automated tests.

### Test Categories

| Category | Priority | Description |
|----------|----------|-------------|
| Endpoint availability | High | Routes respond |
| Request validation | High | Invalid input rejected |
| Response structure | Medium | JSON matches spec |
| Error responses | Medium | Errors formatted correctly |
| Authentication | Medium | Protected endpoints require auth |

### Target Coverage

- Minimum: 15 tests
- Ideal: 20+ tests
- Cover both TicTacToe and PetShop APIs

### Checklist
- [ ] Test framework configured
- [ ] TicTacToe tests written
- [ ] PetShop tests written
- [ ] All tests pass
- [ ] `make test` works

---

## Phase 8: Decision (Go/No-Go)

### Purpose
Decide whether to invest in custom Java generator.

### Decision Matrix

| Question | Criteria |
|----------|----------|
| Can GOAL_MAX.md be met with templates only? | > 80% compliance |
| Is custom generator effort justified? | Strategic importance |
| Is there demand for this framework? | User/market need |
| What's the maintenance burden? | Sustainable long-term |

### Options

1. **Template-Only**: Use existing generator with custom templates
2. **Custom Generator**: Build {framework}-max Java generator
3. **Defer**: Document findings, revisit later
4. **No Support**: Not worth investment

### Decision Document

Create or update ticket with:
- Summary of findings
- Approach comparison
- Effort estimates
- Recommendation
- Next steps

### Checklist
- [ ] All phases completed
- [ ] Decision documented
- [ ] Next steps defined
- [ ] Related tickets updated

---

## Artifact Summary

After completing all phases, you should have:

| Artifact | Location |
|----------|----------|
| Default templates | `openapi-generator-server-templates/openapi-generator-server-{generator}-default/` |
| GENERATOR-ANALYSIS.md | In default templates directory |
| Custom templates | `openapi-generator-server-templates/openapi-generator-server-{generator}/` |
| Generated TicTacToe | `generated/{generator}/tictactoe/` |
| Generated PetShop | `generated/{generator}/petshop/` |
| Demo project | `projects/{framework}-api--{generator}--default/` |
| Integration tests | In demo project `tests/` |
| Decision ticket | `tickets/GENDE-XXX-...` |

---

## Quick Reference: Available Generators

### PHP Generators

| Generator | Framework | Command |
|-----------|-----------|---------|
| php-laravel | Laravel | `-g php-laravel` |
| php-symfony | Symfony | `-g php-symfony` |
| php-slim4 | Slim 4 | `-g php-slim4` |
| php-mezzio-ph | Mezzio | `-g php-mezzio-ph` |
| php-lumen | Lumen | `-g php-lumen` |

### Node.js Generators

| Generator | Framework | Command |
|-----------|-----------|---------|
| typescript-express | Express | `-g typescript-express-server` |
| typescript-node | Node.js | `-g typescript-node` |
| nodejs-express | Express (JS) | `-g nodejs-express-server` |

### Other Languages

| Generator | Language/Framework | Command |
|-----------|-------------------|---------|
| python-flask | Python Flask | `-g python-flask` |
| go-server | Go | `-g go-server` |
| java-spring | Java Spring | `-g spring` |
| kotlin-spring | Kotlin Spring | `-g kotlin-spring` |

### List All Available

```bash
docker run --rm openapitools/openapi-generator-cli:v7.12.0 list
```

---

## References

- [GOAL_MAX.md](../GOAL_MAX.md) - Quality requirements
- [GENERATORS-COMMON.md](./GENERATORS-COMMON.md) - Common template concepts
- [OpenAPI Generator Docs](https://openapi-generator.tech/docs/generators)
- [Mustache Syntax](https://mustache.github.io/mustache.5.html)

---

## Completed Investigations

| Generator | Ticket | Score | Status |
|-----------|--------|-------|--------|
| php-laravel | GENDE-010 | 85% | ✅ laravel-max created |
| php-symfony | GENDE-003 | 54% | ⏸️ Deferred |

*Add new investigations as they are completed.*
