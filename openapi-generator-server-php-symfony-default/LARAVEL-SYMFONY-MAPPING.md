# Laravel to Symfony Component Mapping

**Purpose:** Map Laravel patterns from `laravel-max` to Symfony equivalents for GOAL_MAX.md compliance.

**Target Symfony Version:** Symfony 7.x (LTS 7.4 - November 2025)

**Reference:**
- [Symfony Documentation](https://symfony.com/doc/current/index.html)
- [GOAL_MAX.md](../../GOAL_MAX.md)
- [laravel-max reference](../../generated/laravel-max/)

---

## Overview

| GOAL_MAX Component | Laravel Pattern | Symfony Equivalent | Mapping Difficulty |
|-------------------|-----------------|-------------------|-------------------|
| Routes | PHP routes files | PHP Attributes / YAML | Easy |
| Controllers | Single-action invokable | Single-action controllers | Easy |
| Request Validation | FormRequest | `#[MapRequestPayload]` + DTO | Medium |
| Response Resources | JsonResource | Serializer + Normalizer | Medium |
| Middleware | Middleware classes | Event Listeners / Firewall | Hard |
| Security Interfaces | Middleware interfaces | Authenticator interfaces | Medium |
| DI Configuration | Service Providers | services.yaml / autowiring | Easy |
| Security Validator | Custom validator class | Event Listener + Voters | Hard |

---

## 1. Routes

### Laravel Approach (laravel-max)

```php
// routes/api.php - PHP file with Route facade
$route = $router->POST('/games', \TictactoeApi\Api\Http\Controllers\CreateGameController::class)
    ->name('api.createGame');

// Conditional middleware attachment
if ($router->hasMiddlewareGroup('api.security.createGame')) {
    $route->middleware('api.security.createGame');
}
```

**Key Features:**
- PHP-based route definitions
- Conditional middleware via `hasMiddlewareGroup()`
- Route naming with `->name()`

### Symfony Equivalent

**Option A: PHP Attributes (Recommended for Symfony 7.x)**

```php
// src/Controller/CreateGameController.php
namespace App\Controller;

use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\HttpFoundation\JsonResponse;

#[Route('/games', name: 'api.createGame', methods: ['POST'])]
final class CreateGameController
{
    public function __invoke(/* ... */): JsonResponse
    {
        // ...
    }
}
```

**Option B: YAML Configuration**

```yaml
# config/routes/api.yaml
api.createGame:
    path: /games
    methods: [POST]
    controller: App\Controller\CreateGameController

api.getGame:
    path: /games/{gameId}
    methods: [GET]
    controller: App\Controller\GetGameController
```

### Mapping Notes

| Laravel | Symfony | Notes |
|---------|---------|-------|
| `Route::post()` | `#[Route(methods: ['POST'])]` | Attribute or YAML |
| `->name('api.xxx')` | `name: 'api.xxx'` | Same concept |
| `->middleware()` | Firewall + access_control | Different mechanism |
| Route groups | Route prefix/host | Similar concept |

### Challenges

1. **Conditional middleware** - Laravel's `hasMiddlewareGroup()` pattern has no direct equivalent in Symfony
2. **Route-level middleware** - Symfony uses firewall patterns instead of per-route middleware
3. **Generated routes file** - YAML is more suitable for code generation than PHP attributes

**Recommendation:** Use YAML routes for generated code (easier to generate, similar to current php-symfony output).

---

## 2. Controllers

### Laravel Approach (laravel-max)

```php
// app/Http/Controllers/CreateGameController.php
namespace TictactoeApi\Api\Http\Controllers;

final class CreateGameController
{
    public function __construct(
        private readonly GameManagementApiHandlerInterface $handler
    ) {}

    public function __invoke(CreateGameFormRequest $request): JsonResponse
    {
        $dto = CreateGameRequest::fromArray($request->validated());
        $resource = $this->handler->createGame($dto);
        return $resource->response($request);
    }
}
```

**Key Features:**
- Single-action invokable controller (`__invoke`)
- Constructor injection for handler interface
- FormRequest for validation
- Returns Resource that converts to JsonResponse

### Symfony Equivalent

```php
// src/Controller/CreateGameController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Attribute\MapRequestPayload;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Serializer\SerializerInterface;

#[Route('/games', name: 'api.createGame', methods: ['POST'])]
final class CreateGameController
{
    public function __construct(
        private readonly GameManagementApiHandlerInterface $handler,
        private readonly SerializerInterface $serializer
    ) {}

    public function __invoke(
        #[MapRequestPayload] CreateGameRequest $request
    ): JsonResponse
    {
        $result = $this->handler->createGame($request);

        $json = $this->serializer->serialize($result, 'json');
        return JsonResponse::fromJsonString($json, $result->getStatusCode());
    }
}
```

### Mapping Notes

| Laravel | Symfony | Notes |
|---------|---------|-------|
| `__invoke()` | `__invoke()` | Identical pattern |
| Constructor DI | Constructor DI | Autowiring works same |
| `FormRequest` | `#[MapRequestPayload]` | Different mechanism |
| `$request->validated()` | DTO auto-populated | Symfony handles mapping |
| `$resource->response()` | `JsonResponse` + Serializer | Manual conversion |

### Challenges

1. **Response handling** - Laravel Resources have built-in response conversion; Symfony requires serializer
2. **HTTP status codes** - Need custom DTO/wrapper to carry status code

**Recommendation:** Create response wrapper class that holds data + status code.

---

## 3. Request Validation (FormRequest → DTO + Validator)

### Laravel Approach (laravel-max)

```php
// app/Http/Requests/CreateGameFormRequest.php
namespace TictactoeApi\Api\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

final class CreateGameFormRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'mode' => ['required', 'in:pvp,ai_easy,ai_medium,ai_hard'],
            'opponentId' => ['sometimes', 'string', 'uuid'],
            'isPrivate' => ['sometimes', 'boolean'],
            'metadata' => ['sometimes'],
        ];
    }
}
```

### Symfony Equivalent

```php
// src/Dto/CreateGameRequest.php
namespace App\Dto;

use Symfony\Component\Validator\Constraints as Assert;

final class CreateGameRequest
{
    public function __construct(
        #[Assert\NotBlank]
        #[Assert\Choice(choices: ['pvp', 'ai_easy', 'ai_medium', 'ai_hard'])]
        public readonly string $mode,

        #[Assert\Uuid]
        public readonly ?string $opponentId = null,

        public readonly ?bool $isPrivate = null,

        public readonly ?array $metadata = null,
    ) {}
}
```

**Controller usage:**

```php
public function __invoke(
    #[MapRequestPayload] CreateGameRequest $request
): JsonResponse
```

### Mapping Notes

| Laravel Rule | Symfony Constraint | Notes |
|-------------|-------------------|-------|
| `required` | `#[Assert\NotBlank]` | Equivalent |
| `string` | `#[Assert\Type('string')]` | Or use PHP type |
| `integer` | `#[Assert\Type('int')]` | Or use PHP type |
| `boolean` | `#[Assert\Type('bool')]` | Or use PHP type |
| `in:a,b,c` | `#[Assert\Choice(['a','b','c'])]` | Equivalent |
| `min:5` | `#[Assert\GreaterThanOrEqual(5)]` | For numbers |
| `max:100` | `#[Assert\LessThanOrEqual(100)]` | For numbers |
| `min:5` (string) | `#[Assert\Length(min: 5)]` | For strings |
| `max:100` (string) | `#[Assert\Length(max: 100)]` | For strings |
| `email` | `#[Assert\Email]` | Equivalent |
| `uuid` | `#[Assert\Uuid]` | Equivalent |
| `url` | `#[Assert\Url]` | Equivalent |
| `regex:/pattern/` | `#[Assert\Regex('/pattern/')]` | Equivalent |
| `nullable` | `?type` + no NotBlank | PHP nullable type |
| `sometimes` | Property with default | Optional property |
| `array` | `#[Assert\Type('array')]` | Equivalent |
| `exists:table,col` | Custom Constraint | No built-in |

### Challenges

1. **Database validation** (`exists`, `unique`) - Requires custom constraints in Symfony
2. **Conditional validation** - Symfony uses validation groups
3. **Custom messages** - Both support, different syntax

**Recommendation:** DTOs with Assert attributes work well. Need custom constraints for database validation.

---

## 4. Response Resources (JsonResource → Serializer/Normalizer)

### Laravel Approach (laravel-max)

```php
// app/Http/Resources/CreateGame201Resource.php
namespace TictactoeApi\Api\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;
use TictactoeApi\Model\Game;

final class CreateGame201Resource extends JsonResource
{
    protected int $httpCode = 201;
    public ?string $location = null;

    public function toArray($request): array
    {
        /** @var Game $model */
        $model = $this->resource;
        return [
            'id' => $model->id,
            'status' => $model->status,
            // ... other fields
        ];
    }

    public function withResponse($request, $response): void
    {
        $response->setStatusCode($this->httpCode);
        if ($this->location !== null) {
            $response->header('Location', $this->location);
        }
    }
}
```

**Key Features:**
- Encapsulates HTTP status code
- Encapsulates response headers (Location)
- Transforms model to JSON structure
- Returns `JsonResponse` via `->response()`

### Symfony Equivalent

**Option A: Response Wrapper DTO + Normalizer**

```php
// src/Http/Response/CreateGame201Response.php
namespace App\Http\Response;

final class CreateGame201Response
{
    public function __construct(
        public readonly Game $data,
        public readonly ?string $location = null,
    ) {}

    public function getStatusCode(): int
    {
        return 201;
    }

    public function getHeaders(): array
    {
        $headers = [];
        if ($this->location !== null) {
            $headers['Location'] = $this->location;
        }
        return $headers;
    }
}

// src/Serializer/Normalizer/GameNormalizer.php
namespace App\Serializer\Normalizer;

use Symfony\Component\Serializer\Normalizer\NormalizerInterface;

final class GameNormalizer implements NormalizerInterface
{
    public function normalize(mixed $object, ?string $format = null, array $context = []): array
    {
        /** @var Game $object */
        return [
            'id' => $object->id,
            'status' => $object->status,
            // ... other fields
        ];
    }

    public function supportsNormalization(mixed $data, ?string $format = null, array $context = []): bool
    {
        return $data instanceof Game;
    }
}
```

**Controller usage:**

```php
public function __invoke(#[MapRequestPayload] CreateGameRequest $request): JsonResponse
{
    $response = $this->handler->createGame($request);

    $json = $this->serializer->serialize($response->data, 'json');
    return new JsonResponse(
        $json,
        $response->getStatusCode(),
        $response->getHeaders(),
        true // json already encoded
    );
}
```

**Option B: Custom JsonResponse subclass**

```php
// src/Http/Response/ApiResponse.php
namespace App\Http\Response;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Serializer\SerializerInterface;

abstract class ApiResponse extends JsonResponse
{
    protected int $httpCode = 200;
    protected array $extraHeaders = [];

    public function __construct(
        protected readonly mixed $data,
        SerializerInterface $serializer
    ) {
        $json = $serializer->serialize($data, 'json');
        parent::__construct($json, $this->httpCode, $this->extraHeaders, true);
    }
}

// src/Http/Response/CreateGame201Response.php
final class CreateGame201Response extends ApiResponse
{
    protected int $httpCode = 201;

    public function __construct(Game $data, SerializerInterface $serializer, ?string $location = null)
    {
        if ($location !== null) {
            $this->extraHeaders['Location'] = $location;
        }
        parent::__construct($data, $serializer);
    }
}
```

### Mapping Notes

| Laravel | Symfony | Notes |
|---------|---------|-------|
| `JsonResource` | Normalizer + Response class | Split responsibility |
| `toArray()` | `normalize()` | Same purpose |
| `withResponse()` | Response class methods | Headers/status |
| `->response()` | Manual JsonResponse creation | Different API |

### Challenges

1. **Resource pattern** - Symfony doesn't have a direct equivalent to Laravel's JsonResource
2. **Response encapsulation** - Need custom response wrapper to carry status + headers
3. **Union return types** - Handler interface needs to return response wrappers

**Recommendation:** Create response wrapper classes similar to laravel-max Resources.

---

## 5. Middleware (Laravel Middleware → Symfony Event Listeners / Firewall)

### Laravel Approach (laravel-max)

```php
// Middleware interface
interface BearerHttpAuthenticationInterface
{
    public function handle(Request $request, Closure $next);
}

// Routes with conditional middleware
if ($router->hasMiddlewareGroup('api.security.createGame')) {
    $route->middleware('api.security.createGame');
}
```

**Key Features:**
- Middleware interface per security scheme
- Conditional attachment via middleware groups
- `handle()` method signature

### Symfony Equivalent

**Option A: Custom Authenticator (Security Component)**

```php
// src/Security/BearerHttpAuthenticator.php
namespace App\Security;

use Symfony\Component\Security\Http\Authenticator\AbstractAuthenticator;
use Symfony\Component\Security\Http\Authenticator\Passport\Passport;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class BearerHttpAuthenticator extends AbstractAuthenticator
{
    public function supports(Request $request): ?bool
    {
        return $request->headers->has('Authorization')
            && str_starts_with($request->headers->get('Authorization'), 'Bearer ');
    }

    public function authenticate(Request $request): Passport
    {
        $token = substr($request->headers->get('Authorization'), 7);
        // Validate JWT token and return Passport
        // ...
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        return null; // Continue to controller
    }

    public function onAuthenticationFailure(Request $request, AuthenticationException $exception): ?Response
    {
        return new JsonResponse(['error' => 'Unauthorized'], 401);
    }
}
```

**Security configuration:**

```yaml
# config/packages/security.yaml
security:
    firewalls:
        api:
            pattern: ^/api
            stateless: true
            custom_authenticators:
                - App\Security\BearerHttpAuthenticator

    access_control:
        - { path: ^/api/games, methods: [POST], roles: ROLE_USER }
        - { path: ^/api/games, methods: [GET], roles: PUBLIC_ACCESS }
```

**Option B: Event Listener (for non-auth middleware)**

```php
// src/EventListener/ApiRequestListener.php
namespace App\EventListener;

use Symfony\Component\EventDispatcher\Attribute\AsEventListener;
use Symfony\Component\HttpKernel\Event\RequestEvent;

#[AsEventListener(event: 'kernel.request', priority: 10)]
final class ApiRequestListener
{
    public function __invoke(RequestEvent $event): void
    {
        $request = $event->getRequest();
        // Middleware-like logic here
    }
}
```

### Mapping Notes

| Laravel | Symfony | Notes |
|---------|---------|-------|
| `handle(Request, Closure)` | Authenticator or EventListener | Different pattern |
| Middleware interface | Authenticator interface | Similar contract |
| Middleware groups | Firewall patterns | Configuration-based |
| Per-route middleware | access_control rules | Different granularity |

### Challenges

1. **Per-operation middleware** - Symfony firewall works on URL patterns, not per-route
2. **Multiple auth methods** - Need multiple authenticators and entry points
3. **Conditional attachment** - Laravel's `hasMiddlewareGroup()` has no equivalent
4. **Interface validation** - Symfony doesn't validate middleware implements interface

**Recommendation:** This is the hardest mapping. Use Symfony Security component with custom authenticators and access_control rules.

---

## 6. Security Validator

### Laravel Approach (laravel-max)

```php
// lib/Security/SecurityValidator.php
class SecurityValidator
{
    private static array $securityRequirements = [
        'createGame' => [BearerHttpAuthenticationInterface::class],
        'deleteGame' => [BearerHttpAuthenticationInterface::class],
    ];

    public static function validateMiddleware(Router $router): void
    {
        // Validates middleware is configured correctly
        // Checks: group exists, implements required interface
    }
}
```

### Symfony Equivalent

```php
// src/Security/SecurityConfigValidator.php
namespace App\Security;

use Symfony\Component\EventDispatcher\Attribute\AsEventListener;
use Symfony\Component\HttpKernel\Event\RequestEvent;
use Symfony\Component\HttpKernel\KernelEvents;

#[AsEventListener(event: KernelEvents::REQUEST, priority: 100)]
final class SecurityConfigValidator
{
    private array $securityRequirements = [
        '/api/games' => ['POST' => 'bearerAuth'],
        '/api/games/{id}' => ['DELETE' => 'bearerAuth', 'GET' => 'bearerAuth'],
    ];

    public function __invoke(RequestEvent $event): void
    {
        if (!$event->isMainRequest() || !$this->isDebugMode()) {
            return;
        }

        // Validate security configuration matches requirements
        // Log warnings if misconfigured
    }

    private function isDebugMode(): bool
    {
        return $_ENV['APP_DEBUG'] ?? false;
    }
}
```

### Challenges

1. **Route inspection** - Symfony doesn't easily expose route middleware configuration
2. **Interface checking** - Cannot verify authenticator implements specific interface at runtime
3. **Development-only** - Need to ensure only runs in debug mode

**Recommendation:** Simpler approach - use access_control rules in security.yaml and validate via console command or compile-time check.

---

## 7. Dependency Injection

### Laravel Approach (laravel-max)

```php
// app/Providers/AppServiceProvider.php
public function register(): void
{
    $this->app->bind(
        GameManagementApiHandlerInterface::class,
        GameManagementHandler::class
    );
}
```

### Symfony Equivalent

```yaml
# config/services.yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true

    App\Handler\GameManagementHandler: ~

    App\Handler\GameManagementApiHandlerInterface:
        alias: App\Handler\GameManagementHandler
```

**Or with PHP configuration:**

```php
// config/services.php
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return function (ContainerConfigurator $container): void {
    $services = $container->services();

    $services->alias(
        GameManagementApiHandlerInterface::class,
        GameManagementHandler::class
    );
};
```

### Mapping Notes

| Laravel | Symfony | Notes |
|---------|---------|-------|
| `$app->bind()` | `services.yaml` alias | Configuration-based |
| Service Providers | services.yaml | YAML preferred |
| `$app->singleton()` | `shared: true` (default) | Same concept |
| Auto-discovery | `autoconfigure: true` | Similar |

**Recommendation:** YAML is easier to generate. Symfony autowiring reduces manual configuration.

---

## Summary: Implementation Gaps

### Easy to Implement

| Component | Effort | Notes |
|-----------|--------|-------|
| Routes (YAML) | Low | Current php-symfony already generates this |
| Controllers | Low | Similar pattern |
| DTOs/Models | Low | Current php-symfony already generates this |
| DI Config | Low | services.yaml generation |

### Medium Effort

| Component | Effort | Notes |
|-----------|--------|-------|
| Request DTOs | Medium | Need `#[MapRequestPayload]` + validators |
| Response Wrappers | Medium | Custom response classes needed |
| Handler Interfaces | Medium | Return types need response wrappers |

### Hard to Implement

| Component | Effort | Notes |
|-----------|--------|-------|
| Security (Auth) | High | Authenticators + firewall config |
| Security Validator | High | Different paradigm in Symfony |
| Per-operation middleware | High | Symfony uses URL patterns, not routes |

---

## Conclusion

Creating a `symfony-max` generator matching `laravel-max` quality is **feasible but requires significant architectural decisions**:

1. **Routes:** Use YAML (already supported by php-symfony)
2. **Controllers:** Single-action pattern (needs custom templates)
3. **Validation:** DTOs with Symfony Validator attributes
4. **Responses:** Custom response wrapper classes (new pattern)
5. **Security:** Symfony authenticators + firewall configuration
6. **DI:** services.yaml with interface aliases

**Key difference:** Symfony's security model is more configuration-driven (firewall patterns) vs Laravel's code-driven (middleware groups). The conditional per-operation security in laravel-max doesn't map cleanly to Symfony patterns.

**Recommended approach for GENDE-009:**
- Start with custom templates for php-symfony generator
- Focus on: per-operation controllers, response wrappers, typed handler interfaces
- Security: Generate authenticator stubs + security.yaml patterns
- Accept that some patterns will differ from Laravel due to framework philosophy

---

## References

- [Symfony Routing](https://symfony.com/doc/current/routing.html)
- [Symfony Validation](https://symfony.com/doc/current/validation.html)
- [Symfony Serializer](https://symfony.com/doc/current/serializer.html)
- [Symfony Security](https://symfony.com/doc/current/security.html)
- [Symfony Custom Authenticator](https://symfony.com/doc/current/security/custom_authenticator.html)
- [Symfony 7.3 Security Improvements](https://symfony.com/blog/new-in-symfony-7-3-security-improvements)
- [MapRequestPayload Attribute](https://symfony.com/doc/current/controller.html#mapping-request-payload)
