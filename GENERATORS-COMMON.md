# OpenAPI Generator Common Concepts

This document contains common information shared across all OpenAPI generator analyses. Individual generator analyses reference this document to avoid duplication.

**Reference Date:** 2024-12-22

---

## 1. Template Loop Types

All OpenAPI generators support these mustache loop contexts:

### 1.1 Spec-Level Loops (Once Per Specification)

| Loop | Description | Use Case |
|------|-------------|----------|
| `{{#apiInfo}}` | Entire API specification | Routes file, README, global config |

### 1.2 API-Level Loops (Per Tag/Group)

| Loop | Description | Use Case |
|------|-------------|----------|
| `{{#apis}}` | Per API group (tag) | Iterating through all API groups |
| `{{#operations}}` | Operations in a tag group | Controller/Interface per tag |

### 1.3 Operation-Level Loops (Per Endpoint)

| Loop | Description | Use Case |
|------|-------------|----------|
| `{{#operation}}` | Individual operation | Method generation |
| `{{#pathParams}}` | Path parameters | Route params, method signatures |
| `{{#queryParams}}` | Query parameters | Request parsing |
| `{{#headerParams}}` | Header parameters | Header validation |
| `{{#bodyParams}}` | Body parameters | Request body parsing |
| `{{#formParams}}` | Form parameters | Form data handling |
| `{{#allParams}}` | All parameters | Complete parameter processing |

### 1.4 Model-Level Loops (Per Schema)

| Loop | Description | Use Case |
|------|-------------|----------|
| `{{#models}}` | All models | Iterating through schemas |
| `{{#model}}` | Individual model | DTO/Model class generation |
| `{{#vars}}` | Model properties | Property generation |

### 1.5 Common Conditionals

| Conditional | Description |
|-------------|-------------|
| `{{#required}}` | Parameter/property is required |
| `{{#hasValidation}}` | Has validation rules |
| `{{#maxLength}}` | Max length constraint |
| `{{#minLength}}` | Min length constraint |
| `{{#maximum}}` | Maximum value |
| `{{#minimum}}` | Minimum value |
| `{{#pattern}}` | Regex pattern |
| `{{#maxItems}}` | Max items (arrays) |
| `{{#minItems}}` | Min items (arrays) |
| `{{#isDeprecated}}` | Deprecated flag |
| `{{#isEnum}}` | Is enumeration type |
| `{{#isArray}}` | Is array type |
| `{{#isMap}}` | Is map/dictionary type |

---

## 2. Customization via `files` Config

All generators support custom template configuration via `files` directive:

### 2.1 Configuration Format

```json
{
  "files": {
    "my_template.mustache": {
      "folder": "output/directory",
      "destinationFilename": "{{dynamicName}}.php",
      "templateType": "API|Model|SupportingFiles"
    }
  }
}
```

### 2.2 Template Types

**`"templateType": "API"`**
- Executes **per operations group** (per tag)
- Available loops: `{{#operations}}`, `{{#operation}}`
- Available variables: `{{classname}}`, `{{operationId}}`
- Use for: Interfaces, Controllers, Response Factories (per tag)

**`"templateType": "Model"`**
- Executes **per schema** (components/schemas)
- Available loops: `{{#models}}`, `{{#model}}`, `{{#vars}}`
- Available variables: `{{modelName}}`, `{{classname}}`
- Use for: DTOs, Request objects, Response objects

**`"templateType": "SupportingFiles"`**
- Executes **once per spec**
- Available loops: `{{#apiInfo}}`, `{{#apis}}`
- Use for: Routes, Composer.json, Service Providers, README

### 2.3 Dynamic Filenames

Templates support dynamic filenames using mustache variables:

```json
{
  "destinationFilename": "{{classname}}Controller.php"  // → PetController.php
}
```

Common filename variables:
- `{{classname}}` - Class name from tag (e.g., `Pet`, `User`)
- `{{modelName}}` - Model name from schema
- `{{packageName}}` - Package name from config
- `{{apiVersion}}` - API version

---

## 3. Standard Config Options

Most generators support these configuration options:

| Option | Type | Description |
|--------|------|-------------|
| `invokerPackage` | string | Root namespace |
| `apiPackage` | string | API/Interface package namespace |
| `modelPackage` | string | Model/DTO package namespace |
| `controllerPackage` | string | Controller package namespace |
| `srcBasePath` | string | Source base path (e.g., `lib`) |
| `variableNamingConvention` | string | `camelCase`, `snake_case`, etc. |
| `composerVendorName` | string | Composer vendor name |
| `composerPackageName` | string | Composer package name |
| `gitUserId` | string | Git user ID |
| `gitRepoId` | string | Git repository ID |
| `artifactVersion` | string | Package version |
| `licenseName` | string | License (e.g., `MIT`) |

**Note:** Support varies by generator. Check generator-specific documentation.

---

## 4. PSR-4 Autoloading Requirements

Generated code should follow PSR-4:

**File structure must match namespace:**
```
namespace Vendor\Package\Api;  → src/Api/ClassName.php
```

**Class name must match filename:**
```
File: PetController.php
Class: class PetController { }
```

---

## 5. Generator Architecture Limitations

### 5.1 Per-Tag vs Per-Operation

**Most generators group operations by tag:**
- One file per tag (e.g., `PetApi.php` contains all pet operations)
- Cannot generate one file per operation without custom logic
- `{{classname}}` is based on tag name

**Workaround:**
- Use single-operation tags in OpenAPI spec
- Custom post-processing scripts
- Different generator (e.g., custom generator)

### 5.2 Namespace Customization

Some generators hardcode namespaces:
- Check if generator uses `{{invokerPackage}}` variables
- If hardcoded (e.g., `namespace App\Http\Controllers`), cannot customize without templates

---

## 6. Quality Requirements Mapping

See `/GOAL.md` and `/CLAUDE.md` for our specific requirements.

**Contract Enforcement Requirements:**
1. Interface Contracts → `api.mustache` or custom
2. Request DTOs → `model.mustache` or custom
3. Response Structures → Custom response templates
4. Type Safety → PHP 8.1+ typed properties

**Laravel Framework Structures:**
1. Resources → Custom templates with `JsonResource`
2. Form Requests → Custom templates with `FormRequest`
3. Middleware → Routes configuration
4. Service Providers → `AppServiceProvider.mustache`

**Code Quality:**
1. SOLID → Separate concerns (interface, controller, handler)
2. PSR-4 → Namespace = directory structure
3. DRY → Shared validation, base classes
4. KISS → Simple, readable generated code

---

## 7. References

- OpenAPI Generator Docs: https://openapi-generator.tech/docs/templating
- Mustache Syntax: https://mustache.github.io/mustache.5.html
- PSR-4 Autoloading: https://www.php-fig.org/psr/psr-4/
