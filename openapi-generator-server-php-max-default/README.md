# php-max Default Templates

Default templates for the `php-max` OpenAPI Generator.

## Usage

```bash
# Using default templates (embedded in generator)
openapi-generator generate -g php-max \
  -i spec.yaml \
  -o output/ \
  --additional-properties=invokerPackage=MyApi

# Using these external templates
openapi-generator generate -g php-max \
  -t path/to/openapi-generator-server-php-max-default \
  -i spec.yaml \
  -o output/
```

## Templates

| Template | Description |
|----------|-------------|
| `model.mustache` | PHP classes/enums for OpenAPI schemas |
| `api.mustache` | Service interfaces for API operations |

## Template Variables

### Model Variables

| Variable | Type | Description |
|----------|------|-------------|
| `classname` | string | Class name (PascalCase) |
| `description` | string | Schema description |
| `vars` | array | List of properties |
| `vars[].name` | string | Property name (snake_case) |
| `vars[].baseName` | string | Original property name from spec |
| `vars[].dataType` | string | PHP type |
| `vars[].description` | string | Property description |
| `vars[].required` | boolean | Is property required |
| `vars[].isNullable` | boolean | Is property nullable |
| `vars[].isArray` | boolean | Is property an array |
| `vars[].isEnum` | boolean | Is property an enum |
| `vars[].minLength` | integer | Minimum string length |
| `vars[].maxLength` | integer | Maximum string length |
| `vars[].minimum` | number | Minimum numeric value |
| `vars[].maximum` | number | Maximum numeric value |
| `vars[].pattern` | string | Regex pattern |
| `vendorExtensions.x-is-php-enum` | boolean | Model is a PHP enum |
| `vendorExtensions.x-enum-cases` | array | Enum case definitions |

### API Variables

| Variable | Type | Description |
|----------|------|-------------|
| `classname` | string | API class name |
| `operations` | array | List of operations |
| `operations[].operationId` | string | Operation ID |
| `operations[].summary` | string | Operation summary |
| `operations[].httpMethod` | string | HTTP method |
| `operations[].path` | string | URL path |
| `operations[].allParams` | array | All parameters |
| `operations[].bodyParam` | object | Request body parameter |
| `operations[].responses` | array | Response definitions |
| `imports` | array | Required imports |

### Vendor Extensions (Constraints)

Added by php-max generator for validation:

| Extension | Type | Description |
|-----------|------|-------------|
| `vendorExtensions.hasMinLength` | boolean | Has minLength constraint |
| `vendorExtensions.hasMaxLength` | boolean | Has maxLength constraint |
| `vendorExtensions.hasMinimum` | boolean | Has minimum constraint |
| `vendorExtensions.hasMaximum` | boolean | Has maximum constraint |
| `vendorExtensions.hasPattern` | boolean | Has pattern constraint |
| `vendorExtensions.isUrl` | boolean | Format is url/uri |
| `vendorExtensions.isDate` | boolean | Format is date |
| `vendorExtensions.isDateTime` | boolean | Format is date-time |
| `vendorExtensions.isIpAddress` | boolean | Format is ip/ipv4/ipv6 |
| `vendorExtensions.enumValuesString` | string | Comma-separated enum values |

## Customization

To create framework-specific templates:

1. Copy this directory to a new location
2. Modify templates for your framework
3. Use `-t` flag to specify your template directory

See `openapi-generator-server-php-max-laravel/` for Laravel example.
