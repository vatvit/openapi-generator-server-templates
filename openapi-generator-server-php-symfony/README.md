# Custom php-symfony Templates

**Based on:** OpenAPI Generator 7.12.0 php-symfony templates
**Created:** 2026-01-01

## Overview

These are customized php-symfony templates with improvements over the default templates. They generate Symfony 7.x compatible bundles with better code quality and PSR-12 compliance.

## Improvements Made

### 1. PSR-12 Compliance

All PHP files now have proper PSR-12 file headers:

```php
<?php

declare(strict_types=1);

namespace ...;
```

**Templates updated:**
- `model.mustache`
- `model_generic.mustache`
- `Controller.mustache`
- `api_controller.mustache`
- `api.mustache` (interface)
- `ApiServer.mustache`
- `Bundle.mustache`
- `Extension.mustache`
- `ApiPass.mustache`

### 2. Cleaner Docblocks

Removed outdated and verbose docblock headers. Before:

```php
/**
 * Game
 *
 * PHP version 8.1.1
 *
 * @category Class
 * @package  TictactoeApi
 * @author   OpenAPI Generator team
 * @link     https://github.com/openapitools/openapi-generator
 */
```

After:

```php
/**
 * Game
 *
 * Auto-generated model from OpenAPI specification.
 */
```

### 3. Improved Type Annotations

Added PHPStan-compatible array type annotations:

```php
/** @var array<string, mixed> */
private array $apis = [];
```

Constructor parameter types:

```php
/**
 * @param array<string, mixed>|null $data
 */
public function __construct(?array $data = null)
```

### 4. Alphabetically Sorted Imports

Use statements are now alphabetically sorted per PSR-12:

```php
use JMS\Serializer\Annotation\Accessor;
use JMS\Serializer\Annotation\SerializedName;
use JMS\Serializer\Annotation\Type;
use Symfony\Component\Validator\Constraints as Assert;
```

## Generated Output

### TicTacToe API
```
generated/php-symfony/tictactoe/
├── Api/
│   ├── ApiServer.php
│   ├── GameManagementApiInterface.php
│   ├── GameplayApiInterface.php
│   ├── StatisticsApiInterface.php
│   └── TicTacApiInterface.php
├── Controller/
│   ├── Controller.php
│   ├── GameManagementController.php
│   ├── GameplayController.php
│   ├── StatisticsController.php
│   └── TicTacController.php
├── Model/
│   └── [DTOs with validation]
└── [Bundle structure]
```

### PetShop API
```
generated/php-symfony/petshop/
├── Api/
│   └── [Interfaces]
├── Controller/
│   └── [Controllers]
├── Model/
│   └── [DTOs]
└── [Bundle structure]
```

## Usage

```bash
docker run --rm -v "$(pwd)":/local openapitools/openapi-generator-cli:v7.12.0 generate \
  -g php-symfony \
  -i /local/path/to/openapi.yaml \
  -o /local/generated/php-symfony/output \
  -t /local/openapi-generator-server-templates/openapi-generator-server-php-symfony \
  --additional-properties=invokerPackage=MyApi
```

## Limitations

These improvements are **template-only** changes. The following require a custom Java generator (like `laravel-max`):

| Feature | Status |
|---------|--------|
| Per-operation controllers | Requires Java generator |
| Union return types | Requires Java generator |
| Response DTOs/Resources | Requires Java generator |
| Security middleware pattern | Requires Java generator |

## Comparison with Default Templates

| Aspect | Default | Custom |
|--------|---------|--------|
| `declare(strict_types=1)` | No | Yes |
| Clean docblocks | Verbose | Concise |
| PHPStan array types | No | Yes |
| Sorted imports | No | Yes |
| PSR-12 file headers | No | Yes |

## Files Modified

| Template | Changes |
|----------|---------|
| `model.mustache` | PSR-12 header, clean docblock, sorted imports |
| `model_generic.mustache` | Array type annotation for constructor |
| `Controller.mustache` | PSR-12 header, clean docblock |
| `api_controller.mustache` | PSR-12 header, clean docblock |
| `api.mustache` | PSR-12 header, clean docblock |
| `ApiServer.mustache` | PSR-12 header, array type annotation |
| `Bundle.mustache` | PSR-12 header, sorted imports |
| `Extension.mustache` | PSR-12 header, clean docblock |
| `ApiPass.mustache` | PSR-12 header, clean docblock |

## Related Documentation

- [GENERATOR-ANALYSIS.md](../openapi-generator-server-php-symfony-default/GENERATOR-ANALYSIS.md) - Analysis of default php-symfony generator
- [LARAVEL-SYMFONY-MAPPING.md](../openapi-generator-server-php-symfony-default/LARAVEL-SYMFONY-MAPPING.md) - Laravel to Symfony concept mapping
