# php-max Laravel Templates

Laravel-specific templates for the `php-max` OpenAPI Generator.

## Usage

```bash
openapi-generator generate -g php-max \
  -t path/to/openapi-generator-server-php-max-laravel \
  -i spec.yaml \
  -o output/ \
  --additional-properties=invokerPackage=App\\Generated
```

## Templates

| Template | Description | Output |
|----------|-------------|--------|
| `model.mustache` | PHP DTOs/Enums | `lib/Model/*.php` |
| `api.mustache` | Handler interfaces | `lib/Api/*HandlerInterface.php` |

## Generated Structure

```
output/
└── lib/
    ├── Model/
    │   ├── Game.php           # DTO class
    │   ├── GameStatus.php     # Enum
    │   └── ...
    └── Api/
        ├── GameManagementHandlerInterface.php
        ├── GameplayHandlerInterface.php
        └── ...
```

## Laravel Integration

### 1. Add to composer.json

```json
{
  "autoload": {
    "psr-4": {
      "App\\Generated\\": "generated/lib/"
    }
  }
}
```

### 2. Implement Handler

```php
namespace App\Handlers;

use App\Generated\Api\GameManagementHandlerInterface;
use App\Generated\Model\Game;
use App\Generated\Model\CreateGameRequest;

class GameManagementHandler implements GameManagementHandlerInterface
{
    public function createGame(CreateGameRequest $request): Game
    {
        // Your business logic here
        return new Game(
            id: Str::uuid()->toString(),
            status: GameStatus::Pending,
            // ...
        );
    }
}
```

### 3. Register in ServiceProvider

```php
public function register(): void
{
    $this->app->bind(
        GameManagementHandlerInterface::class,
        GameManagementHandler::class
    );
}
```

## Validation Rules

The templates expose constraint data for building Laravel validation rules.
Use these vendor extensions in your templates:

```mustache
{{#vars}}
'{{baseName}}' => '{{#required}}required{{/required}}{{^required}}nullable{{/required}}{{#vendorExtensions.hasMinLength}}|min:{{minLength}}{{/vendorExtensions.hasMinLength}}{{#vendorExtensions.hasMaxLength}}|max:{{maxLength}}{{/vendorExtensions.hasMaxLength}}{{#isEmail}}|email{{/isEmail}}{{#isUuid}}|uuid{{/isUuid}}{{#vendorExtensions.isUrl}}|url{{/vendorExtensions.isUrl}}',
{{/vars}}
```

## Extending Templates

To add controllers, FormRequests, or Resources:

1. Copy these templates to your project
2. Add new template files
3. The generator currently supports:
   - `model.mustache` - One file per schema
   - `api.mustache` - One file per tag

For per-operation file generation (one controller per endpoint),
additional generator configuration is needed.
