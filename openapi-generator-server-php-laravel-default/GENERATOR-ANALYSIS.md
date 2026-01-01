# PHP-Laravel Generator Analysis

**Generator:** `php-laravel`
**Version:** OpenAPI Generator 7.12.0
**Analysis Date:** 2026-01-01
**Spec Used:** TicTacToe API, PetShop API

## Overview

The `php-laravel` generator produces a Laravel-compatible library with controllers, interfaces, models, routes, and Crell/Serde serialization. It follows Laravel conventions but uses a simpler architecture than full Laravel applications.

## Generated Structure

```
output/
├── routes.php                   # Laravel routes file
├── lib/
│   ├── Api/
│   │   └── *Api.php             # One interface per tag
│   ├── Http/
│   │   └── Controllers/
│   │       └── *Controller.php  # One controller per tag
│   └── Models/
│       └── *.php                # DTOs with Crell/Serde attributes
├── composer.json
├── phpunit.xml
└── README.md
```

## Scoring Against GOAL_MAX.md

| Requirement | Score | Notes |
|-------------|-------|-------|
| **1. Routes File** | ✅ 90% | Clean Laravel routes, proper HTTP methods and paths |
| **2. Controllers** | ⚠️ 55% | One per tag (not per operation), action methods for each operation |
| **3. Middleware Support** | ❌ 10% | No middleware configuration in routes |
| **4. Security Middleware** | ❌ 0% | No security handling at all |
| **5. Validators** | ⚠️ 45% | Inline validation rules in controller, not FormRequest |
| **6. API Interfaces** | ⚠️ 60% | Per-tag interfaces, return type is union of models |
| **7. Response Classes** | ❌ 5% | No response DTOs, just serialized models |
| **8. Response Factories** | ❌ 0% | Not generated |
| **9. DTOs/Models** | ✅ 80% | Good DTOs with typed properties, Crell/Serde attributes |
| **10. Documentation** | ⚠️ 50% | Basic README, no model/API docs |

**Overall Score: 40%**

## Detailed Analysis

### ✅ Strengths

#### 1. Interface-Based Handler Pattern
```php
interface GameManagementApi {
    public function createGame(
        CreateGameRequest $createGameRequest,
    ): Game | BadRequestError | UnauthorizedError;

    public function deleteGame(
        string $gameId,
    ): void | ForbiddenError | NotFoundError;
}
```
Developer implements interface to provide business logic.

#### 2. Model DTOs with Crell/Serde
```php
use Crell\Serde\Renaming\Cases;
use Crell\Serde\Attributes as Serde;

#[Serde\ClassSettings(renameWith: Cases::snake_case)]
class Game
{
    public function __construct(
        public string $id,
        public GameStatus $status,
        public ?string $winner = null,
    ) {}
}
```
Uses modern PHP 8.1+ constructor promotion with proper typing.

#### 3. Clean Routes File
```php
Route::POST('{basePath}/games', [GameManagementController::class, 'createGame'])
    ->name('api.createGame');

Route::DELETE('{basePath}/games/{gameId}', [GameManagementController::class, 'deleteGame'])
    ->name('api.deleteGame');
```
Standard Laravel routing with named routes.

#### 4. Inline Request Validation
```php
$validation = Validator::make($request->all(), [
    'page' => ['integer', 'min:1'],
    'limit' => ['integer', 'min:1', 'max:100'],
]);

if ($validation->fails()) {
    return response()->json(['errors' => $validation->errors()], 400);
}
```

### ❌ Weaknesses

#### 1. No Per-Operation Controllers
All operations in a tag share one controller file. Cannot generate one controller per operation using default templates.

```php
// Default: All operations in ONE controller
class GameManagementController extends Controller
{
    public function createGame(Request $request) { ... }
    public function deleteGame(Request $request, string $gameId) { ... }
    public function getGame(Request $request, string $gameId) { ... }
    public function listGames(Request $request) { ... }
}
```

#### 2. Weak Return Type Safety
```php
public function createGame(...): Game | BadRequestError | UnauthorizedError;
```
While union types are used, they only include model types without HTTP status code context. The controller must determine which status code to return based on the model type.

#### 3. No Response DTOs/Resources
Controllers serialize whatever the handler returns. No generated response classes to enforce schema or HTTP status codes.

```php
if ($apiResult instanceof Game) {
    return response()->json($this->serde->serialize($apiResult, format: 'array'), 201);
}
if ($apiResult instanceof BadRequestError) {
    return response()->json($this->serde->serialize($apiResult, format: 'array'), 400);
}
```

#### 4. No Security Handling
Unlike php-symfony (which has at least method-based security), php-laravel has no security mechanism at all in the generated code.

#### 5. No FormRequest Classes
Uses inline validation in controllers instead of Laravel's FormRequest pattern. Cannot leverage Laravel's authorization or custom validation logic.

#### 6. Method Injection Pattern
```php
public function createGame(Request $request): JsonResponse
{
    // Must instantiate handler manually or rely on service container
}
```
Constructor injection is there but for the interface only, not for other services.

### ⚠️ Limitations for GOAL_MAX.md Compliance

| GOAL_MAX Requirement | php-laravel Support | Gap |
|---------------------|---------------------|-----|
| Contract enforcement via return types | ⚠️ Union of models only | Major - no HTTP status enforcement |
| One controller per operation | ❌ One per tag | Major |
| Response Resources/DTOs | ❌ Not generated | Major |
| Security middleware + validator | ❌ Not supported | Major |
| Per-operation middleware groups | ❌ Not supported | Major |
| FormRequest validation | ❌ Inline only | Medium |

## Comparison with php-symfony

| Aspect | php-laravel | php-symfony |
|--------|-------------|-------------|
| Controller granularity | Per tag | Per tag |
| Interface pattern | ✅ Yes | ✅ Yes |
| Return type safety | ⚠️ Union of models | ❌ `array\|object\|null` |
| Validation | Inline rules | ✅ Symfony Validator |
| Response DTOs | ❌ No | ❌ No |
| DI config | Manual binding | ✅ services.yaml |
| Routing | ✅ routes.php | ✅ routing.yaml |
| Serialization | Crell/Serde | JMS Serializer |
| Security | ❌ None | ⚠️ Method-based |

**Summary:** php-laravel has slightly better return type safety (union types vs `array|object|null`) but worse security handling (none vs method-based). Both share the fundamental limitation of per-tag controller generation.

## Recommendations

### If Customizing Templates

1. **Per-operation controllers** - Modify `api_controller.mustache` to generate one file per operation (achievable with custom templates)
2. **Response DTOs** - Create per-response Resource/Response classes
3. **Security middleware** - Add middleware group pattern to routes
4. **FormRequest validation** - Replace inline validation with FormRequest classes

### Effort Estimate for GOAL_MAX Compliance

| Task | Effort |
|------|--------|
| Custom templates for one controller per operation | Medium |
| Response Resources/Factories | High |
| Security middleware integration | Medium |
| Full GOAL_MAX compliance | **Custom templates can achieve ~85%** |

**Note:** With custom templates, php-laravel can achieve comparable quality to laravel-max (see GENERATOR-COMPARISON.md). The custom templates in `openapi-generator-server-php-laravel/` demonstrate this.

## Conclusion

The `php-laravel` default generator provides a **basic but functional foundation** (40% GOAL_MAX compliance). Its main advantages over php-symfony are:

1. Better model return types (union types vs any)
2. Simpler integration (plain Laravel routing)
3. Modern serialization (Crell/Serde)

Its main disadvantages are:

1. No security handling at all
2. No validation annotations on models
3. No DI configuration file

**Key finding:** The generator's template system is flexible enough that **custom templates can achieve ~85% GOAL_MAX compliance** - see `openapi-generator-server-php-laravel/` for the production-ready custom templates.

For new Laravel projects, consider:
1. **Start with custom templates** (`openapi-generator-server-php-laravel/`)
2. **Or use laravel-max** custom generator for maximum type safety

## Configuration Tips

### Undocumented `controllerPackage` Option

The `controllerPackage` option is **not listed** in `config-help` output but works via config file:

```json
{
  "invokerPackage": "MyApi",
  "apiPackage": "MyApi\\Api",
  "modelPackage": "MyApi\\Model",
  "controllerPackage": "MyApi\\Http\\Controllers"
}
```

Without this, controllers default to `OpenAPI\Server\Http\Controllers` namespace.

**Impact:** Enables multiple APIs in the same project with proper namespace isolation.

### Recommended Config File

```json
{
  "invokerPackage": "TicTacToeApi",
  "apiPackage": "TicTacToeApi\\Api",
  "modelPackage": "TicTacToeApi\\Model",
  "controllerPackage": "TicTacToeApi\\Http\\Controllers",
  "variableNamingConvention": "camelCase"
}
```

## Files Reference

- Default templates: `openapi-generator-server-templates/openapi-generator-server-php-laravel-default/`
- Custom templates: `openapi-generator-server-templates/openapi-generator-server-php-laravel/`
- Demo project: `projects/laravel-api--php-laravel--default/`
- Comparison: `docs/GENERATOR-COMPARISON.md` (laravel-max vs php-laravel with custom templates)
