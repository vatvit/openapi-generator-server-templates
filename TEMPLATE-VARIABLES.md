# php-max Template Variables Reference

This document lists all template variables available in php-max generator templates.

---

## 1. Package/Namespace Variables

Available in ALL templates:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{invokerPackage}}` | Root namespace | `MyApi` |
| `{{modelPackage}}` | Models namespace | `MyApi\Model` |
| `{{apiPackage}}` | API/Service namespace | `MyApi\Api` |
| `{{controllerPackage}}` | Controllers namespace | `MyApi\Controller` |
| `{{handlerPackage}}` | Handlers namespace | `MyApi\Handler` |
| `{{requestPackage}}` | Request DTOs namespace | `MyApi\Request` |
| `{{responsePackage}}` | Response DTOs namespace | `MyApi\Response` |
| `{{securityPackage}}` | Security classes namespace | `MyApi\Security` |
| `{{srcBasePath}}` | Source base path | `lib` |

---

## 2. Model Template Variables

Used in `model.mustache` (per-model loop):

### Model Identity

| Variable | Description | Example |
|----------|-------------|---------|
| `{{classname}}` | Model class name | `CreateGameRequest` |
| `{{description}}` | Model description from spec | `Request to create a game` |
| `{{isEnum}}` | Is this an enum model | `true`/`false` |

### Model Properties

Loop: `{{#vars}}...{{/vars}}`

| Variable | Description | Example |
|----------|-------------|---------|
| `{{name}}` | Property name (snake_case) | `player_name` |
| `{{baseName}}` | Original name from spec | `playerName` |
| `{{dataType}}` | PHP type | `string`, `int`, `array` |
| `{{datatypeWithEnum}}` | Type with enum reference | `GameMode` |
| `{{description}}` | Property description | `Player's display name` |
| `{{required}}` | Is required | `true`/`false` |
| `{{isNullable}}` | Is nullable | `true`/`false` |
| `{{defaultValue}}` | Default value | `null`, `'default'` |

### Property Type Flags

| Variable | Description |
|----------|-------------|
| `{{isString}}` | Is string type |
| `{{isInteger}}` | Is integer type |
| `{{isLong}}` | Is long/int64 type |
| `{{isFloat}}` | Is float type |
| `{{isDouble}}` | Is double type |
| `{{isBoolean}}` | Is boolean type |
| `{{isArray}}` | Is array type |
| `{{isMap}}` | Is map/object type |
| `{{isEnum}}` | Is enum type |
| `{{isDate}}` | Is date format |
| `{{isDateTime}}` | Is date-time format |
| `{{isEmail}}` | Is email format |
| `{{isUuid}}` | Is UUID format |

### Property Constraints

| Variable | Description | Example |
|----------|-------------|---------|
| `{{minLength}}` | Minimum string length | `1` |
| `{{maxLength}}` | Maximum string length | `50` |
| `{{minimum}}` | Minimum numeric value | `0` |
| `{{maximum}}` | Maximum numeric value | `100` |
| `{{exclusiveMinimum}}` | Exclusive minimum | `true`/`false` |
| `{{exclusiveMaximum}}` | Exclusive maximum | `true`/`false` |
| `{{pattern}}` | Regex pattern | `^[a-z]+$` |
| `{{minItems}}` | Minimum array items | `1` |
| `{{maxItems}}` | Maximum array items | `10` |
| `{{uniqueItems}}` | Array items must be unique | `true`/`false` |

### Constraint Presence Flags (vendorExtensions)

| Variable | Description |
|----------|-------------|
| `{{vendorExtensions.hasMinLength}}` | Has minLength constraint |
| `{{vendorExtensions.hasMaxLength}}` | Has maxLength constraint |
| `{{vendorExtensions.hasMinimum}}` | Has minimum constraint |
| `{{vendorExtensions.hasMaximum}}` | Has maximum constraint |
| `{{vendorExtensions.hasPattern}}` | Has pattern constraint |
| `{{vendorExtensions.hasMinItems}}` | Has minItems constraint |
| `{{vendorExtensions.hasMaxItems}}` | Has maxItems constraint |

### Format Flags (vendorExtensions)

| Variable | Description |
|----------|-------------|
| `{{vendorExtensions.isUrl}}` | Is URL/URI format |
| `{{vendorExtensions.isDate}}` | Is date format |
| `{{vendorExtensions.isDateTime}}` | Is date-time format |
| `{{vendorExtensions.isIpAddress}}` | Is IP address format |

### Enum Handling

| Variable | Description | Example |
|----------|-------------|---------|
| `{{vendorExtensions.enumValuesString}}` | Comma-separated values | `pvp,ai_easy,ai_hard` |
| `{{vendorExtensions.enumValues}}` | List of enum values | `['pvp', 'ai_easy']` |
| `{{vendorExtensions.x-is-php-enum}}` | Is PHP 8.1+ enum | `true` |
| `{{vendorExtensions.x-enum-cases}}` | Enum cases list | See below |

Enum cases loop: `{{#vendorExtensions.x-enum-cases}}...{{/vendorExtensions.x-enum-cases}}`

| Variable | Description | Example |
|----------|-------------|---------|
| `{{name}}` | PHP case name | `AiEasy` |
| `{{value}}` | Original value | `ai_easy` |
| `{{isString}}` | Is string-backed | `true` |

---

## 3. API Template Variables

Used in `api.mustache` (per-tag loop):

| Variable | Description | Example |
|----------|-------------|---------|
| `{{classname}}` | API class name (includes "Api") | `GameManagementApi` |
| `{{classVarName}}` | Variable name | `gameManagementApi` |
| `{{description}}` | Tag description | `Game management operations` |

### Operations Loop

`{{#operations}}{{#operation}}...{{/operation}}{{/operations}}`

See Operation Variables below.

---

## 4. Operation Template Variables

Used in `controller.mustache`, `handler.mustache`, etc. (per-operation):

### Operation Identity

| Variable | Description | Example |
|----------|-------------|---------|
| `{{operationId}}` | Operation ID | `createGame` |
| `{{operationIdCamelCase}}` | camelCase | `createGame` |
| `{{operationIdPascalCase}}` | PascalCase | `CreateGame` |
| `{{classname}}` | Class name for file | `CreateGameController` |
| `{{summary}}` | Short summary | `Create a new game` |
| `{{notes}}` | Detailed description | `Creates a TicTacToe game...` |
| `{{httpMethod}}` | HTTP method | `POST` |
| `{{path}}` | URL path | `/games` |

### Operation Object

Access via `{{operation.fieldName}}`:

| Variable | Description |
|----------|-------------|
| `{{operation.operationId}}` | Operation ID |
| `{{operation.baseName}}` | Tag/API base name (without "Api") |
| `{{operation.summary}}` | Summary |
| `{{operation.httpMethod}}` | HTTP method |
| `{{operation.path}}` | Path |

### HTTP Method Flags

| Variable | Description |
|----------|-------------|
| `{{isGet}}` | Is GET request |
| `{{isPost}}` | Is POST request |
| `{{isPut}}` | Is PUT request |
| `{{isPatch}}` | Is PATCH request |
| `{{isDelete}}` | Is DELETE request |

### Parameter Collections

| Variable | Description |
|----------|-------------|
| `{{allParams}}` | All parameters |
| `{{pathParams}}` | Path parameters |
| `{{queryParams}}` | Query parameters |
| `{{headerParams}}` | Header parameters |
| `{{formParams}}` | Form parameters |
| `{{bodyParam}}` | Request body parameter |

### Parameter Presence Flags

| Variable | Description |
|----------|-------------|
| `{{hasBodyParam}}` | Has request body |
| `{{hasPathParams}}` | Has path parameters |
| `{{hasQueryParams}}` | Has query parameters |
| `{{hasHeaderParams}}` | Has header parameters |
| `{{hasFormParams}}` | Has form parameters |

### Parameter Loop Variables

`{{#allParams}}...{{/allParams}}` or `{{#pathParams}}...{{/pathParams}}` etc.

| Variable | Description | Example |
|----------|-------------|---------|
| `{{paramName}}` | Parameter name | `gameId` |
| `{{baseName}}` | Original name | `game_id` |
| `{{dataType}}` | PHP type | `string` |
| `{{description}}` | Description | `Game identifier` |
| `{{required}}` | Is required | `true`/`false` |
| `{{isPathParam}}` | Is path parameter | `true`/`false` |
| `{{isQueryParam}}` | Is query parameter | `true`/`false` |
| `{{isHeaderParam}}` | Is header parameter | `true`/`false` |
| `{{isBodyParam}}` | Is body parameter | `true`/`false` |
| `{{defaultValue}}` | Default value | `null` |

Parameters have same constraint variables as properties (minLength, maxLength, etc.)

### Body Parameter Properties

When `{{hasBodyParam}}` is true, access body properties:

`{{#bodyParam}}{{#vars}}...{{/vars}}{{/bodyParam}}`

Properties have same variables as model properties.

### Responses

`{{#responses}}...{{/responses}}`

| Variable | Description | Example |
|----------|-------------|---------|
| `{{code}}` | HTTP status code | `200` |
| `{{message}}` | Response description | `Successful response` |
| `{{dataType}}` | Response type | `Game` |
| `{{vendorExtensions.isSuccess}}` | Is 2xx response | `true` |
| `{{vendorExtensions.isError}}` | Is non-2xx response | `false` |
| `{{vendorExtensions.is2xx}}` | Is 2xx | `true` |
| `{{vendorExtensions.is4xx}}` | Is 4xx | `false` |
| `{{vendorExtensions.is5xx}}` | Is 5xx | `false` |

---

## 5. Security Variables

### Global Security Schemes

Available in all templates:

| Variable | Description |
|----------|-------------|
| `{{hasSecuritySchemes}}` | Has any security schemes |
| `{{securitySchemes}}` | List of security schemes |

Security schemes loop: `{{#securitySchemes}}...{{/securitySchemes}}`

| Variable | Description | Example |
|----------|-------------|---------|
| `{{name}}` | Scheme name | `bearerAuth` |
| `{{classname}}` | Class name | `BearerAuth` |
| `{{type}}` | Scheme type | `http`, `apiKey`, `oauth2` |
| `{{description}}` | Description | `JWT Bearer token` |

Type-specific flags:

| Variable | Description |
|----------|-------------|
| `{{isHttp}}` | Is HTTP auth |
| `{{isBearer}}` | Is Bearer token |
| `{{isBasic}}` | Is Basic auth |
| `{{isApiKey}}` | Is API key |
| `{{isApiKeyInHeader}}` | API key in header |
| `{{isApiKeyInQuery}}` | API key in query |
| `{{isApiKeyInCookie}}` | API key in cookie |
| `{{isOAuth2}}` | Is OAuth2 |
| `{{isOpenIdConnect}}` | Is OpenID Connect |

### Per-Operation Security

| Variable | Description |
|----------|-------------|
| `{{hasAuthMethods}}` | Operation requires auth |
| `{{authMethods}}` | Auth methods for operation |
| `{{vendorExtensions.securitySchemeNames}}` | List of scheme names |
| `{{vendorExtensions.securitySchemesString}}` | Comma-separated names |

---

## 6. Supporting File Variables

Used in `routes.mustache`, `composer.json.mustache`, etc.

### All Operations

| Variable | Description |
|----------|-------------|
| `{{hasOperations}}` | Has any operations |
| `{{allOperations}}` | List of all operations |

Loop: `{{#allOperations}}...{{/allOperations}}`

Each item has all operation variables listed above.

---

## 7. Loop Control Variables

Available in any loop:

| Variable | Description |
|----------|-------------|
| `{{-first}}` | Is first item |
| `{{-last}}` | Is last item |
| `{{-index}}` | Zero-based index |

Example:
```mustache
{{#allParams}}
    ${{paramName}}{{^-last}},{{/-last}}
{{/allParams}}
```

---

## 8. files.json Configuration

Template output is configured via `files.json`:

```json
{
  "templates": {
    "model": {
      "template": "model.mustache",
      "folder": "Model",
      "suffix": ".php",
      "enabled": true
    },
    "api": {
      "template": "api.mustache",
      "folder": "Api",
      "suffix": "ServiceInterface.php",
      "enabled": true
    },
    "controller": {
      "template": "controller.mustache",
      "folder": "Controller",
      "suffix": "Controller.php",
      "enabled": true,
      "condition": null
    },
    "formrequest": {
      "template": "formrequest.mustache",
      "folder": "Request",
      "suffix": "FormRequest.php",
      "enabled": true,
      "condition": "hasBodyParam"
    }
  },
  "supporting": [
    { "template": "routes.mustache", "output": "routes/api.php" },
    { "template": "composer.json.mustache", "output": "composer.json" }
  ]
}
```

### Conditions

| Condition | Description |
|-----------|-------------|
| `null` | Always generate |
| `hasBodyParam` | Only if operation has body |
| `hasQueryParams` | Only if operation has query params |
| `hasPathParams` | Only if operation has path params |
| `hasFormParams` | Only if operation has form params |
| `hasHeaderParams` | Only if operation has header params |

### Empty Templates

To disable file generation, make template empty:
```mustache
{{! Empty template - no files generated }}
```
