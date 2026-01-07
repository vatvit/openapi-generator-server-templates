# php-max Symfony Templates

Symfony framework templates for the `php-max` OpenAPI Generator.

## Quick Start

```bash
# Generate Symfony library with these templates
openapi-generator generate -g php-max \
  -i your-api-spec.yaml \
  -o ./generated/my-api \
  -t path/to/openapi-generator-server-php-max-symfony \
  --additional-properties=invokerPackage=MyApi
```

## What Gets Generated

| File | Description |
|------|-------------|
| `src/Model/*.php` | DTO classes from OpenAPI schemas |
| `src/Api/*Api.php` | Service classes - one per API tag |
| `src/Controller/*.php` | Symfony controllers |
| `config/routes.yaml` | Route definitions |
| `config/services.yaml` | Dependency injection configuration |
| `composer.json` | Package configuration |

## Generated Library Structure

```
generated/my-api/
├── src/
│   ├── Api/
│   │   └── PetApi.php              # API service class
│   ├── Controller/
│   │   └── PetController.php       # Symfony controller
│   └── Model/
│       └── Pet.php                 # DTO class
├── config/
│   ├── routes.yaml
│   └── services.yaml
└── composer.json
```

## Requirements

- PHP 8.1+
- Symfony 6.x or 7.x

## Integration into Symfony Project

1. **Add as dependency**:
   ```json
   {
     "repositories": [
       { "type": "path", "url": "../generated/my-api" }
     ],
     "require": {
       "my-vendor/my-api": "*"
     }
   }
   ```

2. **Import routes** in `config/routes.yaml`:
   ```yaml
   my_api:
       resource: '../vendor/my-vendor/my-api/config/routes.yaml'
   ```

3. **Import services** in `config/services.yaml`:
   ```yaml
   imports:
       - { resource: '../vendor/my-vendor/my-api/config/services.yaml' }
   ```

---

## Template Files

| Template | Purpose |
|----------|---------|
| `api.mustache` | API service class |
| `controller.mustache` | Symfony controller |
| `model.mustache` | Model dispatcher (routes to model_generic or model_enum) |
| `model_generic.mustache` | DTO class template |
| `model_enum.mustache` | PHP 8.1+ enum template |
| `routes.yaml.mustache` | Symfony route definitions |
| `services.yaml.mustache` | DI container configuration |
| `composer.json.mustache` | Composer package definition |
| `files.json` | Template configuration |

## Related Template Sets

- **php-max-default** - Laravel templates (default for php-max generator)
- **php-max-slim** - Slim Framework templates
