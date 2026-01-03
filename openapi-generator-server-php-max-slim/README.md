# php-max Slim Templates

Slim Framework templates for the php-max OpenAPI generator.

## Generated Structure

```
lib/
├── Model/                     # DTOs and enums
│   ├── Pet.php
│   └── CreatePetRequest.php
├── Service/                   # Service interfaces (per-tag)
│   └── PetsApiServiceInterface.php
├── Handler/                   # PSR-15 handlers (per-operation)
│   ├── CreatePetHandler.php
│   └── GetPetHandler.php
├── routes.php                 # Route definitions
├── dependencies.php           # DI container config
└── composer.json
```

## Architecture

- **Per-operation handlers** - One PSR-15 RequestHandler per API operation
- **Per-tag service interfaces** - Business logic contracts grouped by tag
- **PSR-7/PSR-15** - Standard HTTP message interfaces
- **PHP-DI** - Dependency injection container

## Usage

### 1. Generate Code

```bash
openapi-generator generate \
  -g php-max \
  -i openapi.yaml \
  -o ./generated \
  -t path/to/openapi-generator-server-php-max-slim
```

### 2. Install Dependencies

```bash
cd generated
composer install
```

### 3. Implement Service Interfaces

```php
namespace App\Service;

use Generated\Service\PetsApiServiceInterface;
use Generated\Model\Pet;
use Generated\Model\CreatePetRequest;

class PetsService implements PetsApiServiceInterface
{
    public function createPet(CreatePetRequest $body): mixed
    {
        // Your business logic here
        return ['id' => 1, 'name' => $body->name];
    }

    public function getPet(string $petId): mixed
    {
        // Your business logic here
        return ['id' => $petId, 'name' => 'Fluffy'];
    }
}
```

### 4. Configure DI Container

Update `dependencies.php` with your implementations:

```php
return [
    PetsApiServiceInterface::class => \DI\autowire(
        \App\Service\PetsService::class
    ),
    // ... handler autowiring stays as generated
];
```

### 5. Setup Slim Application

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use DI\ContainerBuilder;
use Slim\Factory\AppFactory;

// Build container
$containerBuilder = new ContainerBuilder();
$containerBuilder->addDefinitions(__DIR__ . '/generated/dependencies.php');
$container = $containerBuilder->build();

// Create app with container
AppFactory::setContainer($container);
$app = AppFactory::create();

// Add routes
(require __DIR__ . '/generated/routes.php')($app, $container);

// Add middleware
$app->addBodyParsingMiddleware();
$app->addRoutingMiddleware();
$app->addErrorMiddleware(true, true, true);

$app->run();
```

## Template Files

| File | Purpose |
|------|---------|
| `model.mustache` | DTOs and PHP 8.1+ enums |
| `api.mustache` | Service interfaces (per-tag) |
| `controller.mustache` | PSR-15 handlers (per-operation) |
| `routes.mustache` | Slim route definitions |
| `dependencies.mustache` | PHP-DI container config |
| `composer.json.mustache` | Package dependencies |
| `files.json` | Template configuration |

## Requirements

- PHP 8.2+
- Slim Framework 4.x
- PHP-DI 7.x
- PSR-7 implementation (slim/psr7)
