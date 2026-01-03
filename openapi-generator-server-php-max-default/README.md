# php-max Default Templates (Laravel)

The default template set for the `php-max` OpenAPI Generator produces a **Laravel-compatible PHP library**.

## Quick Start

```bash
# Generate Laravel library with default templates
openapi-generator generate -g php-max \
  -i your-api-spec.yaml \
  -o ./generated/my-api \
  -t path/to/openapi-generator-server-php-max-default \
  --additional-properties=invokerPackage=MyApi
```

## What Gets Generated

| File | Description |
|------|-------------|
| `app/Models/*.php` | DTO classes and PHP 8.1+ enums from OpenAPI schemas |
| `app/Handlers/*Interface.php` | Service interfaces - one per API tag |
| `app/Http/Controllers/*.php` | Laravel controllers - one per operation |
| `app/Http/Requests/*.php` | Laravel Form Requests with validation rules |
| `routes/api.php` | Route definitions for all operations |
| `app/Providers/*ServiceProvider.php` | Service provider for dependency injection |
| `composer.json` | Package configuration |

## Generated Library Structure

```
generated/my-api/
├── app/
│   ├── Handlers/
│   │   └── PetApiHandlerInterface.php    # Interface for business logic
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── CreatePetController.php   # One controller per operation
│   │   │   └── GetPetController.php
│   │   └── Requests/
│   │       └── CreatePetFormRequest.php  # Validation rules from spec
│   ├── Models/
│   │   ├── Pet.php                       # DTO class
│   │   └── PetStatus.php                 # PHP 8.1+ enum
│   └── Providers/
│       └── MyApiServiceProvider.php
├── routes/
│   └── api.php
└── composer.json
```

## Integration into Laravel Project

1. **Add as dependency** (composer path or packagist):
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

2. **Implement the handler interface**:
   ```php
   // app/Services/PetHandler.php
   class PetHandler implements PetApiHandlerInterface
   {
       public function createPet(CreatePetFormRequest $request): Pet
       {
           // Your business logic here
           return new Pet(id: 1, name: $request->name);
       }
   }
   ```

3. **Register in service provider**:
   ```php
   // app/Providers/AppServiceProvider.php
   $this->app->bind(PetApiHandlerInterface::class, PetHandler::class);
   ```

4. **Include routes**:
   ```php
   // routes/api.php
   require base_path('vendor/my-vendor/my-api/routes/api.php');
   ```

---

## Alternative Template Sets

### Symfony Templates

For Symfony projects, use the `php-max-symfony` template set:

```bash
openapi-generator generate -g php-max \
  -i your-api-spec.yaml \
  -o ./generated/my-api \
  -t path/to/openapi-generator-server-php-max-symfony \
  --additional-properties=invokerPackage=MyApi
```

**Symfony-specific features:**
- Symfony Validator attributes (`#[Assert\NotNull]`, `#[Assert\Uuid]`, etc.)
- `services.yaml` for dependency injection
- `routes.yaml` for Symfony routing
- Symfony Bundle structure

### Creating Custom Framework Templates

To create templates for another framework (Slim, Laminas, CodeIgniter, etc.):

1. **Copy the default templates**:
   ```bash
   cp -r openapi-generator-server-php-max-default openapi-generator-server-php-max-myframework
   ```

2. **Modify templates** for your framework:
   - `controller.mustache` - Adapt to framework's controller pattern
   - `routes.mustache` - Use framework's routing syntax
   - `model.mustache` - Add framework-specific attributes/annotations
   - `files.json` - Define which files to generate and where

3. **Update `files.json`** to control output paths:
   ```json
   {
     "files": [
       {
         "templateFile": "controller.mustache",
         "destinationFilename": "src/Controller/{{classname}}Controller.php",
         "templateType": "Operation"
       }
     ]
   }
   ```

4. **Use your templates**:
   ```bash
   openapi-generator generate -g php-max \
     -t path/to/openapi-generator-server-php-max-myframework \
     ...
   ```

---

## Template Files Reference

### model.mustache

Generates DTO classes and PHP 8.1+ enums.

**Key variables:**
| Variable | Description |
|----------|-------------|
| `classname` | Class name (PascalCase) |
| `vars` | Array of properties |
| `vars[].name` | Property name |
| `vars[].dataType` | PHP type |
| `vars[].required` | Is required |
| `vars[].isNullable` | Is nullable |
| `vars[].isArray` | Is array type |
| `vars[].isMap` | Is associative array (object with additionalProperties) |
| `vendorExtensions.x-is-php-enum` | Generate as PHP enum |
| `vendorExtensions.x-enum-cases` | Enum cases with name/value |

### api.mustache

Generates handler interfaces (one per API tag).

**Key variables:**
| Variable | Description |
|----------|-------------|
| `classname` | Interface name |
| `operations` | Array of operations in this tag |
| `operations[].operationId` | Method name |
| `operations[].allParams` | All parameters |
| `operations[].bodyParam` | Request body parameter |
| `operations[].returnType` | Response type |

### controller.mustache

Generates Laravel controllers (one per operation).

**Key variables:**
| Variable | Description |
|----------|-------------|
| `operationId` | Operation/method name |
| `httpMethod` | GET, POST, PUT, DELETE, etc. |
| `path` | URL path with parameters |
| `pathParams` | Path parameters |
| `queryParams` | Query parameters |
| `bodyParam` | Request body |
| `responses` | Response definitions |

### formrequest.mustache

Generates Laravel Form Request classes with validation.

**Validation variables:**
| Variable | Description |
|----------|-------------|
| `vars[].required` | `'required'` rule |
| `vars[].isString` | `'string'` rule |
| `vars[].isInteger` | `'integer'` rule |
| `vars[].isBoolean` | `'boolean'` rule |
| `vars[].isArray` | `'array'` rule |
| `vars[].minLength` | `'min:N'` rule |
| `vars[].maxLength` | `'max:N'` rule |
| `vars[].minimum` | `'min:N'` rule (numeric) |
| `vars[].maximum` | `'max:N'` rule (numeric) |
| `vars[].pattern` | `'regex:/.../` rule |
| `vars[].isEmail` | `'email'` rule |
| `vars[].isUuid` | `'uuid'` rule |
| `vars[].isUri` | `'url'` rule |

### routes.mustache

Generates Laravel route definitions.

### provider.mustache

Generates Laravel Service Provider for package auto-discovery.

### files.json

Controls which templates generate which files:

```json
{
  "files": [
    {
      "templateFile": "model.mustache",
      "destinationFilename": "app/Models/{{classname}}.php",
      "templateType": "Model"
    },
    {
      "templateFile": "controller.mustache",
      "destinationFilename": "app/Http/Controllers/{{operationId}}Controller.php",
      "templateType": "Operation"
    }
  ]
}
```

**Template types:**
- `Model` - Generated once per schema
- `API` - Generated once per tag (group of operations)
- `Operation` - Generated once per operation
- `SupportingFiles` - Generated once per generation run

---

## Configuration Options

### Additional Properties

| Property | Default | Description |
|----------|---------|-------------|
| `invokerPackage` | `OpenAPI` | Root namespace |
| `modelPackage` | `Model` | Models sub-namespace |
| `apiPackage` | `Api` | API interfaces sub-namespace |
| `srcBasePath` | `src` | Source directory |

### Example with all options

```bash
openapi-generator generate -g php-max \
  -i spec.yaml \
  -o ./output \
  -t ./templates/php-max-default \
  --additional-properties=invokerPackage=Acme\\PetStore \
  --additional-properties=modelPackage=Acme\\PetStore\\Model \
  --additional-properties=apiPackage=Acme\\PetStore\\Api
```

---

## Comparison: Default (Laravel) vs Symfony

| Feature | Default (Laravel) | Symfony |
|---------|-------------------|---------|
| Validation | Form Request classes | `#[Assert\*]` attributes |
| Routing | `routes/api.php` | `routes.yaml` |
| DI Config | Service Provider | `services.yaml` |
| Controllers | Laravel Controllers | Symfony Controllers |
| Models | Plain DTOs | DTOs with Validator attributes |

Choose based on your target framework.
