# PHP-Symfony Generator Analysis

**Generator:** `php-symfony`
**Version:** OpenAPI Generator 7.12.0
**Analysis Date:** 2026-01-01
**Spec Used:** TicTacToe API

## Overview

The `php-symfony` generator produces a Symfony Bundle with controllers, interfaces, models, routing, and DI configuration. It follows Symfony conventions and uses JMS Serializer for JSON handling.

## Generated Structure

```
output/
├── Api/
│   ├── ApiServer.php              # Handler registry
│   └── *ApiInterface.php          # One interface per tag
├── Controller/
│   ├── Controller.php             # Base controller
│   └── *Controller.php            # One controller per tag
├── Model/
│   └── *.php                      # DTOs with JMS/Validator annotations
├── Service/
│   ├── SerializerInterface.php
│   ├── JmsSerializer.php
│   ├── ValidatorInterface.php
│   └── SymfonyValidator.php
├── DependencyInjection/
│   ├── OpenAPIServerExtension.php
│   └── Compiler/ApiPass.php
├── Resources/config/
│   ├── routing.yaml               # Symfony routing
│   └── services.yaml              # DI configuration
├── Tests/                         # PHPUnit tests
├── docs/                          # API documentation
├── OpenAPIServerBundle.php        # Symfony Bundle
└── composer.json
```

## Scoring Against GOAL_MAX.md

| Requirement | Score | Notes |
|-------------|-------|-------|
| **1. Routes File** | ✅ 90% | YAML routing config, proper HTTP methods and paths |
| **2. Controllers** | ⚠️ 60% | One per tag (not per operation), action methods for each operation |
| **3. Middleware Support** | ❌ 20% | No middleware concept, security via method calls |
| **4. Security Middleware** | ❌ 30% | Interface method `setbearerHttpAuthentication`, no middleware/validator |
| **5. Validators** | ✅ 85% | Symfony Validator with Assert attributes on models |
| **6. API Interfaces** | ⚠️ 70% | Per-tag interfaces, but return type is `array\|object\|null` |
| **7. Response Classes** | ❌ 20% | No response DTOs, raw serialization |
| **8. Response Factories** | ❌ 0% | Not generated |
| **9. DTOs/Models** | ✅ 85% | Good DTOs with typed properties, JMS annotations |
| **10. Documentation** | ✅ 80% | Markdown docs for models and APIs |

**Overall Score: 54%**

## Detailed Analysis

### ✅ Strengths

#### 1. Interface-Based Handler Pattern
```php
interface GameManagementApiInterface
{
    public function createGame(
        CreateGameRequest $createGameRequest,
        int &$responseCode,
        array &$responseHeaders
    ): array|object|null;
}
```
Developer implements interface to provide business logic.

#### 2. Model DTOs with Validation
```php
class Game
{
    #[Assert\NotNull]
    #[Assert\Type("string")]
    protected ?string $id = null;

    #[Assert\NotNull]
    #[Assert\Valid]
    protected ?GameStatus $status = null;
}
```
Uses Symfony Validator attributes for validation.

#### 3. Symfony Bundle Structure
- Proper bundle organization
- DI configuration via services.yaml
- Routing via routing.yaml
- Compiler pass for API registration

#### 4. Request Validation in Controllers
```php
$asserts = [];
$asserts[] = new Assert\NotNull();
$asserts[] = new Assert\Type("int");
$asserts[] = new Assert\GreaterThanOrEqual(1);
$response = $this->validate($page, $asserts);
```

### ❌ Weaknesses

#### 1. No Per-Operation Controllers
All operations in a tag share one controller file. Cannot generate one controller per operation using `files` config.

#### 2. Weak Return Type Safety
```php
public function createGame(...): array|object|null;
```
Return type `array|object|null` provides no compile-time contract enforcement. Developer can return anything.

#### 3. Response Code/Headers by Reference
```php
public function createGame(
    CreateGameRequest $createGameRequest,
    int &$responseCode,           // ← By reference
    array &$responseHeaders       // ← By reference
): array|object|null;
```
This pattern is awkward and error-prone. No type safety for response.

#### 4. No Response DTOs/Resources
Controllers serialize whatever the handler returns. No generated response classes to enforce schema.

#### 5. Security via Method, Not Middleware
```php
$handler->setbearerHttpAuthentication($securitybearerHttpAuthentication);
```
Security token passed to handler method instead of proper middleware chain. No enforcement mechanism.

#### 6. JMS Serializer (Legacy)
Uses JMS Serializer instead of Symfony Serializer component. JMS is older and less integrated with modern Symfony.

#### 7. Namespace Bug
```php
?\TictactoeApi\Model\TictactoeApi\Model\GameStatus $status
```
Double namespace issue in generated code.

### ⚠️ Limitations for GOAL_MAX.md Compliance

| GOAL_MAX Requirement | php-symfony Support | Gap |
|---------------------|---------------------|-----|
| Contract enforcement via return types | ❌ Returns `array\|object\|null` | Major |
| One controller per operation | ❌ One per tag | Major |
| Response Resources/DTOs | ❌ Not generated | Major |
| Security middleware + validator | ❌ Method-based only | Major |
| Per-operation middleware groups | ❌ Not supported | Major |

## Comparison with php-laravel

| Aspect | php-symfony | php-laravel |
|--------|-------------|-------------|
| Controller granularity | Per tag | Per tag |
| Interface pattern | ✅ Yes | ✅ Yes |
| Return type safety | ❌ `array\|object\|null` | ❌ Similar |
| Validation | ✅ Symfony Validator | ✅ Laravel Validator |
| Response DTOs | ❌ No | ❌ No |
| DI config | ✅ services.yaml | ⚠️ Manual |
| Routing | ✅ routing.yaml | ✅ routes.php |
| Serialization | JMS Serializer | Crell/Serde |

Both generators have similar limitations for GOAL_MAX.md compliance.

## Recommendations

### If Customizing Templates

1. **Response DTOs** - Create per-response Resource classes
2. **Type-safe returns** - Use union types for specific response types
3. **Per-operation controllers** - Would require custom generator (like laravel-max)
4. **Security middleware** - Use Symfony Event Listeners instead of method injection

### Effort Estimate for GOAL_MAX Compliance

| Task | Effort |
|------|--------|
| Custom templates for existing generator | Medium |
| Per-operation file generation | Requires custom Java generator |
| Full GOAL_MAX compliance | High (similar to laravel-max effort) |

## Conclusion

The `php-symfony` generator provides a solid foundation but has the same fundamental limitation as `php-laravel`: **cannot generate per-operation files without a custom Java generator**.

For Symfony support matching laravel-max quality, we would likely need to create a `symfony-max` custom generator following the same approach.

## Files Reference

- Templates: `openapi-generator-server-templates/openapi-generator-server-php-symfony-default/`
- Generated sample: `generated/php-symfony/tictactoe/`
