# Template Authoring Guide

This guide explains how to create and customize templates for the `laravel-max` OpenAPI Generator.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Project Structure](#project-structure)
3. [Creating a New Generator](#creating-a-new-generator)
4. [Template System](#template-system)
5. [Available Variables](#available-variables)
6. [Adding New File Types](#adding-new-file-types)
7. [Testing Your Generator](#testing-your-generator)
8. [Common Patterns](#common-patterns)

---

## Architecture Overview

The generator consists of two main components:

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenAPI Specification                     │
│                      (tictactoe.json)                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Java Generator Class                        │
│              (LaravelMaxGenerator.java)                      │
│                                                              │
│  • Extends AbstractPhpCodegen                                │
│  • Processes OpenAPI spec into template variables            │
│  • Controls file generation (what files, where)              │
│  • Adds custom logic (e.g., security schemes, validation)    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Mustache Templates                         │
│              (src/main/resources/laravel-max/)               │
│                                                              │
│  • model.mustache → Models/DTOs                              │
│  • controller.mustache → Http/Controllers                    │
│  • resource.mustache → Http/Resources                        │
│  • form-request.mustache → Http/Requests                     │
│  • api-interface.mustache → Handlers (interfaces)            │
│  • routes.mustache → routes/api.php                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Generated PHP Code                        │
│                  (generated/tictactoe/)                      │
└─────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Java for Logic, Mustache for Structure**: Complex logic (loops, conditionals, data transformation) goes in Java. Templates handle code structure and formatting.

2. **One File Per Purpose**: Each template generates a specific type of file (models, controllers, etc.)

3. **Variables are Pre-computed**: The Java generator prepares all variables before template rendering. Templates just insert values.

---

## Project Structure

```
laravel-max-generator/
├── pom.xml                           # Maven build configuration
├── src/
│   ├── main/
│   │   ├── java/org/openapitools/codegen/laravelmax/
│   │   │   └── LaravelMaxGenerator.java    # Main generator class
│   │   └── resources/
│   │       ├── META-INF/services/
│   │       │   └── org.openapitools.codegen.CodegenConfig  # Service registration
│   │       └── laravel-max/                # Template directory
│   │           ├── model.mustache
│   │           ├── controller.mustache
│   │           ├── resource.mustache
│   │           ├── form-request.mustache
│   │           ├── api-interface.mustache
│   │           ├── routes.mustache
│   │           ├── security-interface.mustache
│   │           ├── security-validator.mustache
│   │           ├── query-params.mustache
│   │           ├── error-resource.mustache
│   │           └── licenseInfo.mustache    # Partial for file headers
│   └── test/
│       └── java/.../LaravelMaxGeneratorTest.java
└── target/
    └── laravel-max-openapi-generator-1.0.0.jar  # Built JAR
```

---

## Creating a New Generator

### Step 1: Scaffold with OpenAPI Generator Meta

```bash
# Create a new generator scaffold
docker run --rm -v $(pwd):/local openapitools/openapi-generator-cli meta \
  -o /local/my-generator \
  -n my-framework \
  -p org.openapitools.codegen.myframework
```

### Step 2: Choose Base Class

In your generator Java class, extend the appropriate base class:

| Language | Base Class | Use When |
|----------|------------|----------|
| PHP | `AbstractPhpCodegen` | Laravel, Symfony, etc. |
| TypeScript | `AbstractTypeScriptCodegen` | Express, NestJS, etc. |
| JavaScript | `AbstractJavaScriptCodegen` | Node.js vanilla |
| Java | `AbstractJavaCodegen` | Spring, etc. |

```java
public class MyFrameworkGenerator extends AbstractPhpCodegen implements CodegenConfig {
    // ...
}
```

### Step 3: Configure Generator Properties

```java
public MyFrameworkGenerator() {
    super();

    // Generator metadata
    embeddedTemplateDir = templateDir = "my-framework";

    // Output structure
    setModelPackage("App\\Models");
    setApiPackage("App");
    setInvokerPackage("MyApi");

    // Source folder (relative to output)
    setSourceFolder("app");

    // Type mappings (OpenAPI types → PHP types)
    typeMapping.put("DateTime", "\\DateTime");
    typeMapping.put("date", "\\DateTime");
}
```

### Step 4: Build and Test

```bash
# Build JAR
docker run --rm -v $(pwd):/app -w /app maven:3.9-eclipse-temurin-17 \
  mvn clean package -DskipTests

# Generate code
docker run --rm -v $(pwd):/app -w /app maven:3.9-eclipse-temurin-17 \
  mvn generate-sources -f generate-pom.xml
```

---

## Template System

### Mustache Basics

Mustache is a logic-less template language. Key syntax:

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{variable}}` | Insert value (HTML-escaped) | `{{classname}}` → `CreateGameRequest` |
| `{{{variable}}}` | Insert value (unescaped) | `{{{dataType}}}` → `\DateTime` |
| `{{#section}}...{{/section}}` | Conditional/Loop | `{{#hasParams}}...{{/hasParams}}` |
| `{{^section}}...{{/section}}` | Inverted (if NOT) | `{{^required}}?{{/required}}` |
| `{{>partial}}` | Include another template | `{{>licenseInfo}}` |
| `{{! comment }}` | Comment (not rendered) | `{{! TODO: fix this }}` |

### Template Example

```mustache
<?php declare(strict_types=1);

{{>licenseInfo}}
namespace {{{modelPackage}}};

/**
 * {{classname}} DTO
 */
class {{classname}}
{
{{#vars}}
    public {{#isNullable}}?{{/isNullable}}{{{dataType}}} ${{nameInCamelCase}}{{#defaultValue}} = {{{defaultValue}}}{{/defaultValue}};
{{/vars}}

    public function __construct(
{{#vars}}
        {{#isNullable}}?{{/isNullable}}{{{dataType}}} ${{nameInCamelCase}}{{#defaultValue}} = {{{defaultValue}}}{{/defaultValue}},
{{/vars}}
    ) {
{{#vars}}
        $this->{{nameInCamelCase}} = ${{nameInCamelCase}};
{{/vars}}
    }
}
```

### Triple Braces for PHP

Always use `{{{variable}}}` (triple braces) for:
- Namespaces: `{{{modelPackage}}}` → `App\Models`
- Type hints: `{{{dataType}}}` → `\DateTime`
- Default values: `{{{defaultValue}}}` → `'all-time'`

Double braces `{{variable}}` will HTML-encode backslashes and quotes!

---

## Available Variables

### Global Variables

Available in all templates:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{invokerPackage}}` | Root namespace | `TictactoeApi` |
| `{{apiPackage}}` | API namespace | `TictactoeApi` |
| `{{modelPackage}}` | Models namespace | `TictactoeApi\Models` |
| `{{appVersion}}` | API version | `1.0.0` |

### Model Variables

Available in `model.mustache`:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{classname}}` | Model class name | `CreateGameRequest` |
| `{{description}}` | Schema description | `Request to create a game` |
| `{{#vars}}` | Loop over properties | - |
| `{{name}}` | Property name (original) | `player_id` |
| `{{nameInCamelCase}}` | Property name (camelCase) | `playerId` |
| `{{baseName}}` | JSON field name | `player_id` |
| `{{dataType}}` | PHP type | `string`, `int`, `\DateTime` |
| `{{required}}` | Is required? | `true`/`false` |
| `{{isNullable}}` | Is nullable? | `true`/`false` |
| `{{defaultValue}}` | Default value | `'all-time'`, `10` |
| `{{isDateTime}}` | Is DateTime type? | `true`/`false` |
| `{{isEnum}}` | Is enum type? | `true`/`false` |
| `{{isArray}}` | Is array type? | `true`/`false` |

### Operation Variables

Available in controller/resource templates:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{operationId}}` | Operation ID | `createGame` |
| `{{operationIdCamelCase}}` | PascalCase | `CreateGame` |
| `{{httpMethod}}` | HTTP method | `POST` |
| `{{path}}` | API path | `/games/{gameId}` |
| `{{summary}}` | Operation summary | `Create a new game` |
| `{{notes}}` | Operation description | `Creates a TicTacToe game...` |
| `{{#pathParams}}` | Path parameters | - |
| `{{#queryParams}}` | Query parameters | - |
| `{{#bodyParam}}` | Request body | - |
| `{{#responses}}` | Response definitions | - |

### Response Variables

Available in resource templates:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{code}}` | HTTP status code | `201` |
| `{{message}}` | Status message | `Created` |
| `{{#schema}}` | Response schema | - |
| `{{#isArray}}` | Is array response? | `true`/`false` |

### Custom Variables (Vendor Extensions)

The generator adds custom variables via `vendorExtensions`:

```java
// In Java generator
op.vendorExtensions.put("x-custom-var", "value");

// In template
{{#vendorExtensions.x-custom-var}}
  Custom value: {{vendorExtensions.x-custom-var}}
{{/vendorExtensions.x-custom-var}}
```

---

## Adding New File Types

### Step 1: Create Template

Create `my-file.mustache` in `src/main/resources/laravel-max/`:

```mustache
<?php declare(strict_types=1);

namespace {{{invokerPackage}}}\MyNamespace;

class {{classname}}
{
    // Generated code
}
```

### Step 2: Register in Java Generator

Add a method to generate files using your template:

```java
private void writeMyFiles() {
    // Load template
    String templatePath = "laravel-max/my-file.mustache";
    String templateContent = readResourceContents(templatePath);

    // Compile template (disable HTML escaping for PHP)
    com.samskivert.mustache.Template template =
        com.samskivert.mustache.Mustache.compiler()
            .escapeHTML(false)
            .compile(templateContent);

    // Prepare data
    Map<String, Object> data = new HashMap<>();
    data.put("classname", "MyClass");
    data.put("invokerPackage", invokerPackage);

    // Render
    String content = template.execute(data);

    // Write file
    String outputPath = outputFolder + "/app/MyNamespace/MyClass.php";
    writeToFile(outputPath, content);
}
```

### Step 3: Call from postProcessOperationsWithModels

```java
@Override
public OperationsMap postProcessOperationsWithModels(OperationsMap objs, List<ModelMap> allModels) {
    OperationsMap results = super.postProcessOperationsWithModels(objs, allModels);

    // Generate custom files
    writeMyFiles();

    return results;
}
```

---

## Testing Your Generator

### Unit Tests

Create tests in `src/test/java/`:

```java
public class MyGeneratorTest extends TestCase {

    @Test
    public void testGeneratedFilesExist() {
        // Generate code
        File output = generate("my-spec.yaml");

        // Verify files exist
        assertTrue(new File(output, "app/Models/Game.php").exists());
        assertTrue(new File(output, "app/Http/Controllers/CreateGameController.php").exists());
    }

    @Test
    public void testGeneratedCodeSyntax() {
        File output = generate("my-spec.yaml");

        // Run PHP lint on all files
        // ...
    }
}
```

### Integration Tests

Create a Laravel test project in `test-integration/`:

```
test-integration/
├── app/
├── tests/
│   └── Feature/
│       └── CreateGameControllerTest.php
├── composer.json
└── phpunit.xml
```

Run tests:

```bash
docker run --rm \
  -v $(pwd)/test-integration:/app \
  -v $(pwd)/../generated/tictactoe:/app/../../generated/tictactoe \
  -w /app \
  php:8.4-cli php vendor/bin/phpunit --testdox
```

### PHP Linting

Always lint generated PHP files:

```bash
docker run --rm \
  -v $(pwd)/generated/tictactoe:/code \
  php:8.4-cli \
  sh -c "find /code -name '*.php' -exec php -l {} \;"
```

---

## Common Patterns

### Handling Optional Properties

```mustache
{{! Nullable type hint for optional properties }}
public {{^required}}?{{/required}}{{{dataType}}} ${{nameInCamelCase}}{{^required}} = null{{/required}};
```

### DateTime Handling

```mustache
{{#isDateTime}}
    {{nameInCamelCase}}: isset($data['{{baseName}}'])
        ? new \DateTime($data['{{baseName}}'])
        : null,
{{/isDateTime}}
{{^isDateTime}}
    {{nameInCamelCase}}: $data['{{baseName}}'] ?? null,
{{/isDateTime}}
```

### Enum Validation Rules

```java
// In Java generator
if (prop.isEnum && prop.allowableValues != null) {
    List<String> values = (List<String>) prop.allowableValues.get("values");
    String inRule = "in:" + String.join(",", values);
    rules.add(inRule);
}
```

### Union Return Types

```mustache
public function {{operationId}}(
{{#allParams}}
    {{{dataType}}} ${{paramName}}{{^-last}},{{/-last}}
{{/allParams}}
): {{#responses}}{{operationIdCamelCase}}{{code}}Resource{{^-last}}|{{/-last}}{{/responses}};
```

### Conditional Middleware

```mustache
{{#hasMiddlewareGroup}}
Route::middleware([{{{middlewareGroup}}}])->group(function () {
{{/hasMiddlewareGroup}}
    Route::{{httpMethod}}('{{{path}}}', {{controllerClass}}::class);
{{#hasMiddlewareGroup}}
});
{{/hasMiddlewareGroup}}
```

---

## Debugging Tips

### 1. Dump All Variables

Create a debug template to see all available variables:

```mustache
{{! debug.mustache }}
classname: {{classname}}
vars:
{{#vars}}
  - name: {{name}}
    nameInCamelCase: {{nameInCamelCase}}
    dataType: {{dataType}}
    required: {{required}}
    isNullable: {{isNullable}}
{{/vars}}
```

### 2. Java Logging

Add logging in the generator:

```java
LOGGER.info("Processing operation: " + op.operationId);
LOGGER.info("Path params: " + op.pathParams);
LOGGER.info("Body param: " + (op.bodyParam != null ? op.bodyParam.dataType : "none"));
```

### 3. Template Compilation Errors

If templates fail to compile, check for:
- Unmatched `{{#section}}` / `{{/section}}`
- Typos in variable names
- Missing closing braces

---

## Reference

### Files Generated by laravel-max

| Template | Output Path | Count | Description |
|----------|-------------|-------|-------------|
| model.mustache | app/Models/*.php | Per schema | DTOs with fromArray/toArray |
| controller.mustache | app/Http/Controllers/*.php | Per operation | Invokable controllers |
| resource.mustache | app/Http/Resources/*.php | Per response | JsonResource with HTTP code |
| form-request.mustache | app/Http/Requests/*.php | Per POST/PUT op | Validation rules |
| api-interface.mustache | app/Handlers/*.php | Per tag | Handler interfaces |
| routes.mustache | routes/api.php | 1 | Route definitions |
| security-interface.mustache | app/Security/*.php | Per security scheme | Auth interfaces |
| security-validator.mustache | app/Security/SecurityValidator.php | 1 | Middleware validator |
| query-params.mustache | app/Models/*QueryParams.php | Per GET op with params | Query DTOs |
| error-resource.mustache | app/Http/Resources/*ErrorResource.php | Per error code | Error responses |

### Useful Links

- [OpenAPI Generator Documentation](https://openapi-generator.tech/docs/customization)
- [Mustache Manual](https://mustache.github.io/mustache.5.html)
- [AbstractPhpCodegen Source](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator/src/main/java/org/openapitools/codegen/languages/AbstractPhpCodegen.java)
