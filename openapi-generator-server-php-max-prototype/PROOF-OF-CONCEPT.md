# Pure Template Solution: Multi-Framework PHP Support

**Purpose:** Demonstrate that the same OpenAPI specification can produce code for multiple PHP frameworks using only template differences.

**Conclusion:** **YES, a pure template solution IS FEASIBLE** for all five frameworks analyzed.

**Investigation Date:** 2026-01-02

---

## Executive Summary

| Framework | Feasibility | Complexity | Validation Approach | Controller Type |
|-----------|-------------|------------|---------------------|-----------------|
| **Laravel** | ✅ Fully feasible | Medium | FormRequest rules array | Invokable controller |
| **Symfony** | ✅ Fully feasible | Medium | Assert attributes on DTO | Attribute-routed controller |
| **Slim** | ✅ Fully feasible | Low | Respect/Validation chains | PSR-15 handler |
| **Laminas** | ✅ Fully feasible | Medium | InputFilter with validators | PSR-15 handler |
| **CodeIgniter** | ✅ Fully feasible | Low | Pipe-delimited rules | RESTful controller |

All five frameworks can be supported from the **same OpenAPI Generator** using **different template sets**. The generator provides identical template variables; only the formatting differs.

---

## 1. Core Insight: Same Variables, Different Formatting

OpenAPI Generator passes the **SAME template variables** to all templates:

**Constraint Variables:**
- `{{required}}`, `{{maxLength}}`, `{{minLength}}`, `{{pattern}}`
- `{{minimum}}`, `{{maximum}}`, `{{exclusiveMinimum}}`, `{{exclusiveMaximum}}`
- `{{maxItems}}`, `{{minItems}}`, `{{uniqueItems}}`

**Type Variables:**
- `{{isString}}`, `{{isInteger}}`, `{{isLong}}`, `{{isFloat}}`, `{{isDouble}}`
- `{{isBoolean}}`, `{{isArray}}`, `{{isMap}}`, `{{isEnum}}`, `{{isDate}}`, `{{isDateTime}}`
- `{{isEmail}}`, `{{isUuid}}`, `{{isUri}}`

**Operation Variables:**
- `{{operationId}}`, `{{classname}}`, `{{httpMethod}}`, `{{path}}`
- `{{summary}}`, `{{notes}}`, `{{hasAuthMethods}}`, `{{authMethods}}`

**The only difference is HOW templates format this data.**

---

## 2. Five-Framework Validation Comparison

### OpenAPI Specification (Source)

```yaml
CreateGameRequest:
  type: object
  required:
    - mode
  properties:
    mode:
      type: string
      enum: [pvp, ai_easy, ai_medium, ai_hard]
    playerName:
      type: string
      minLength: 1
      maxLength: 50
    maxPlayers:
      type: integer
      minimum: 2
      maximum: 10
    email:
      type: string
      format: email
```

### Laravel Output (FormRequest rules array)

```php
public function rules(): array
{
    return [
        'mode' => ['required', 'string', 'in:pvp,ai_easy,ai_medium,ai_hard'],
        'playerName' => ['sometimes', 'string', 'min:1', 'max:50'],
        'maxPlayers' => ['sometimes', 'integer', 'min:2', 'max:10'],
        'email' => ['sometimes', 'string', 'email'],
    ];
}
```

### Symfony Output (Property attributes)

```php
#[Assert\NotBlank]
#[Assert\Choice(['pvp', 'ai_easy', 'ai_medium', 'ai_hard'])]
public readonly string $mode;

#[Assert\Length(min: 1, max: 50)]
public readonly ?string $playerName = null;

#[Assert\GreaterThanOrEqual(2)]
#[Assert\LessThanOrEqual(10)]
public readonly ?int $maxPlayers = null;

#[Assert\Email]
public readonly ?string $email = null;
```

### Slim Output (Respect/Validation)

```php
use Respect\Validation\Validator as v;

$modeValidator = v::stringType()->in(['pvp', 'ai_easy', 'ai_medium', 'ai_hard']);
$playerNameValidator = v::optional(v::stringType()->length(1, 50));
$maxPlayersValidator = v::optional(v::intType()->min(2)->max(10));
$emailValidator = v::optional(v::email());
```

### Laminas Output (InputFilter with Validators)

```php
$this->add([
    'name' => 'mode',
    'required' => true,
    'validators' => [
        ['name' => NotEmpty::class],
        [
            'name' => InArray::class,
            'options' => ['haystack' => ['pvp', 'ai_easy', 'ai_medium', 'ai_hard']],
        ],
    ],
]);

$this->add([
    'name' => 'playerName',
    'required' => false,
    'validators' => [
        [
            'name' => StringLength::class,
            'options' => ['min' => 1, 'max' => 50],
        ],
    ],
]);

$this->add([
    'name' => 'maxPlayers',
    'required' => false,
    'validators' => [
        [
            'name' => Between::class,
            'options' => ['min' => 2, 'max' => 10],
        ],
    ],
]);

$this->add([
    'name' => 'email',
    'required' => false,
    'validators' => [
        ['name' => EmailAddress::class],
    ],
]);
```

### CodeIgniter 4 Output (Pipe-delimited rules)

```php
$rules = [
    'mode' => 'required|string|in_list[pvp,ai_easy,ai_medium,ai_hard]',
    'playerName' => 'permit_empty|string|min_length[1]|max_length[50]',
    'maxPlayers' => 'permit_empty|integer|greater_than_equal_to[2]|less_than_equal_to[10]',
    'email' => 'permit_empty|valid_email',
];
```

**All five derive from the SAME template variables!**

---

## 3. Template Variable Mapping (5 Frameworks)

| Variable | Laravel | Symfony | Slim | Laminas | CodeIgniter |
|----------|---------|---------|------|---------|-------------|
| `{{#required}}` | `'required'` | `#[NotBlank]` | (no wrap) | `required: true` | `required` |
| `{{^required}}` | `'sometimes'` | `?type` | `optional()` | `required: false` | `permit_empty` |
| `{{#maxLength}}` | `'max:N'` | `Length(max:N)` | `->length(,N)` | `StringLength max` | `max_length[N]` |
| `{{#minLength}}` | `'min:N'` | `Length(min:N)` | `->length(N,)` | `StringLength min` | `min_length[N]` |
| `{{#minimum}}` | `'min:N'` | `GreaterThanOrEqual` | `->min(N)` | `Between min` | `greater_than_equal_to[N]` |
| `{{#maximum}}` | `'max:N'` | `LessThanOrEqual` | `->max(N)` | `Between max` | `less_than_equal_to[N]` |
| `{{#pattern}}` | `'regex:P'` | `Regex('/P/')` | `->regex()` | `Regex pattern` | `regex_match[P]` |
| `{{#isEnum}}` | `'in:a,b'` | `Choice([...])` | `->in([])` | `InArray` | `in_list[a,b]` |
| `{{#isEmail}}` | `'email'` | `Email` | `->email()` | `EmailAddress` | `valid_email` |
| `{{#isUuid}}` | `'uuid'` | `Uuid` | `->uuid()` | `Uuid` | custom rule |
| `{{#isUri}}` | `'url'` | `Url` | `->url()` | `Uri` | `valid_url` |

---

## 4. Framework Architecture Comparison

| Aspect | Laravel | Symfony | Slim | Laminas | CodeIgniter |
|--------|---------|---------|------|---------|-------------|
| **Philosophy** | Full-stack | Component-based | Micro | Enterprise PSR | Lightweight |
| **Validation** | FormRequest | DTO Attributes | External lib | InputFilter | Built-in rules |
| **Controller** | `Controller` | `AbstractController` | PSR-15 Handler | PSR-15 Handler | `ResourceController` |
| **Response** | `JsonResource` | Serializer | PSR-7 | PSR-7 JsonResponse | ResponseTrait |
| **Routing** | PHP facade | YAML/Attributes | PHP chain | PHP config | PHP array |
| **DI Container** | Laravel DI | Symfony DI | PSR-11 | PSR-11 | Built-in |
| **Middleware** | Laravel MW | Event/Firewall | PSR-15 MW | PSR-15 MW | CI Filters |

### Complexity Assessment

| Framework | Validation | Routing | Controllers | DI Setup | Security | Total |
|-----------|------------|---------|-------------|----------|----------|-------|
| **Laravel** | Easy | Easy | Easy | Easy | Medium | **Medium** |
| **Symfony** | Medium | Easy | Easy | Easy | Hard | **Medium** |
| **Slim** | Easy | Easy | Easy | Medium | Easy | **Low** |
| **Laminas** | Medium | Easy | Easy | Medium | Medium | **Medium** |
| **CodeIgniter** | Easy | Easy | Easy | Easy | Easy | **Low** |

---

## 5. Template Directory Structure

```
openapi-generator-server-php-max/
├── common/                              # Shared templates
│   ├── model_enum.mustache             # Enums (identical across frameworks)
│   ├── php_header.mustache             # File headers
│   └── model_base.mustache             # Base model structure
│
├── laravel/                             # Laravel 10+
│   ├── validation_rules.mustache       # FormRequest rules array
│   ├── form_request.mustache           # FormRequest class
│   ├── controller.mustache             # Invokable controller
│   ├── routes.mustache                 # PHP routes
│   ├── resource.mustache               # JsonResource
│   └── service_provider.mustache       # DI configuration
│
├── symfony/                             # Symfony 6+
│   ├── validation_attrs.mustache       # Assert attributes
│   ├── dto.mustache                    # Request DTO
│   ├── controller.mustache             # Attribute-routed controller
│   ├── routing.mustache                # YAML routes
│   ├── normalizer.mustache             # Serializer normalizer
│   └── services.mustache               # services.yaml
│
├── slim/                                # Slim 4
│   ├── validation_respect.mustache     # Respect/Validation rules
│   ├── validator.mustache              # Validator class
│   ├── handler.mustache                # PSR-15 handler
│   ├── routes.mustache                 # Slim routes
│   └── container.mustache              # PHP-DI definitions
│
├── laminas/                             # Laminas Mezzio
│   ├── input_filter.mustache           # InputFilter class
│   ├── handler.mustache                # PSR-15 handler
│   ├── routes.mustache                 # Mezzio routes config
│   └── config_provider.mustache        # ConfigProvider for DI
│
├── codeigniter/                         # CodeIgniter 4
│   ├── validation_rules.mustache       # Pipe-delimited rules
│   ├── controller.mustache             # RESTful controller
│   ├── routes.mustache                 # CI4 routes config
│   └── service.mustache                # Service class
│
└── configs/
    ├── laravel.json                    # Laravel file generation
    ├── symfony.json                    # Symfony file generation
    ├── slim.json                       # Slim file generation
    ├── laminas.json                    # Laminas file generation
    └── codeigniter.json                # CodeIgniter file generation
```

---

## 6. Validation Syntax Quick Reference

### Required Field

| Framework | Syntax |
|-----------|--------|
| Laravel | `'required'` |
| Symfony | `#[Assert\NotBlank]` |
| Slim | no `optional()` wrapper |
| Laminas | `'required' => true` + `NotEmpty` validator |
| CodeIgniter | `'required'` |

### String Length (min: 1, max: 50)

| Framework | Syntax |
|-----------|--------|
| Laravel | `'min:1', 'max:50'` |
| Symfony | `#[Assert\Length(min: 1, max: 50)]` |
| Slim | `->length(1, 50)` |
| Laminas | `StringLength::class, ['min' => 1, 'max' => 50]` |
| CodeIgniter | `'min_length[1]\|max_length[50]'` |

### Numeric Range (min: 2, max: 10)

| Framework | Syntax |
|-----------|--------|
| Laravel | `'min:2', 'max:10'` |
| Symfony | `#[GreaterThanOrEqual(2)]`, `#[LessThanOrEqual(10)]` |
| Slim | `->min(2)->max(10)` |
| Laminas | `Between::class, ['min' => 2, 'max' => 10]` |
| CodeIgniter | `'greater_than_equal_to[2]\|less_than_equal_to[10]'` |

### Enum Values (a, b, c)

| Framework | Syntax |
|-----------|--------|
| Laravel | `'in:a,b,c'` |
| Symfony | `#[Assert\Choice(['a', 'b', 'c'])]` |
| Slim | `->in(['a', 'b', 'c'])` |
| Laminas | `InArray::class, ['haystack' => ['a', 'b', 'c']]` |
| CodeIgniter | `'in_list[a,b,c]'` |

---

## 7. Prototype Files Created

```
openapi-generator-server-php-max-prototype/
├── PROOF-OF-CONCEPT.md              # This document
│
├── laravel/
│   ├── validation_rules.mustache    # ✅ Created
│   └── controller.mustache          # ✅ Created
│
├── symfony/
│   ├── validation_attrs.mustache    # ✅ Created
│   └── controller.mustache          # ✅ Created
│
├── slim/
│   ├── validation_respect.mustache  # ✅ Created
│   ├── handler.mustache             # ✅ Created
│   └── routes.mustache              # ✅ Created
│
├── laminas/
│   ├── input_filter.mustache        # ✅ Created
│   ├── handler.mustache             # ✅ Created
│   └── routes.mustache              # ✅ Created
│
└── codeigniter/
    ├── validation_rules.mustache    # ✅ Created
    ├── controller.mustache          # ✅ Created
    └── routes.mustache              # ✅ Created
```

---

## 8. Challenges and Solutions

### Challenge 1: Validation Syntax Varies Widely

| Framework | Style |
|-----------|-------|
| Laravel | String array with pipes |
| Symfony | PHP 8 Attributes |
| Slim | Method chaining |
| Laminas | Array configuration |
| CodeIgniter | Pipe-delimited string |

**Solution:** Separate validation templates per framework using same variables.

### Challenge 2: Combined vs Separate Constraints

**Example:** minLength + maxLength

| Framework | Combined Support |
|-----------|-----------------|
| Laravel | Separate only: `'min:1', 'max:50'` |
| Symfony | Combined: `#[Length(min: 1, max: 50)]` |
| Slim | Combined: `->length(1, 50)` |
| Laminas | Combined: `['min' => 1, 'max' => 50]` |
| CodeIgniter | Separate: `'min_length[1]\|max_length[50]'` |

**Solution:** Template conditionals handle combinations where beneficial.

### Challenge 3: PSR-7/15 vs Framework-Specific

| Framework | HTTP Abstraction |
|-----------|-----------------|
| Laravel | Custom Request/Response |
| Symfony | HttpFoundation |
| Slim | PSR-7 |
| Laminas | PSR-7 |
| CodeIgniter | Custom (PSR-7 compatible) |

**Solution:** Each template uses framework-native abstractions.

### Challenge 4: Security Patterns

| Framework | Security Approach |
|-----------|------------------|
| Laravel | Middleware classes |
| Symfony | Firewall + Voters |
| Slim | PSR-15 Middleware |
| Laminas | PSR-15 Middleware + ACL |
| CodeIgniter | Filters |

**Solution:** Generate security stubs appropriate for each framework.

---

## 9. Implementation Options

### Option A: Single Generator + Template Sets (RECOMMENDED)

```bash
# Same generator, different template directories
openapi-generator generate -g php-max -t templates/laravel ...
openapi-generator generate -g php-max -t templates/symfony ...
openapi-generator generate -g php-max -t templates/slim ...
openapi-generator generate -g php-max -t templates/laminas ...
openapi-generator generate -g php-max -t templates/codeigniter ...
```

**Pros:** Single Java codebase, templates fully control output, easy to add frameworks
**Cons:** Requires custom generator development

### Option B: Configuration-Driven Selection

```json
{
  "framework": "laravel",  // or symfony, slim, laminas, codeigniter
  "templateSet": "auto"    // auto-selects based on framework
}
```

**Pros:** Best UX, single config option
**Cons:** Requires Java changes

---

## 10. Effort Estimate

| Task | Days |
|------|------|
| Base generator setup | 2-3 |
| Laravel templates (complete) | 3-4 |
| Symfony templates (complete) | 3-4 |
| Slim templates (complete) | 2-3 |
| Laminas templates (complete) | 3-4 |
| CodeIgniter templates (complete) | 2-3 |
| Testing & documentation | 3-4 |
| **Total** | **18-25** |

---

## 11. Conclusion

**A pure template-based solution IS FEASIBLE for ALL FIVE frameworks:**

### Key Findings

1. **Same Variables:** All frameworks use identical template variables from OpenAPI Generator
2. **Different Formatting:** Templates control framework-specific output format
3. **File Control:** JSON `files` config determines which templates generate which files
4. **Scalability:** Adding new frameworks = adding new template directory (no Java changes)

### Validation Pattern Summary

| Pattern Type | Frameworks |
|--------------|------------|
| **String array** | Laravel |
| **Attributes** | Symfony |
| **Method chains** | Slim (Respect) |
| **Array config** | Laminas |
| **Pipe string** | CodeIgniter |

### Recommended Approach

**Option A: Single Generator + Template Sets**

1. Create or extend `php-max` generator that passes raw OpenAPI data
2. Create template directories: `laravel/`, `symfony/`, `slim/`, `laminas/`, `codeigniter/`
3. Use `files` config to control file generation per framework
4. Share common templates (enums, headers) via `common/` directory

---

## 12. References

### External Documentation
- [Laravel Validation](https://laravel.com/docs/validation)
- [Symfony Validation](https://symfony.com/doc/current/validation.html)
- [Slim 4 Documentation](https://www.slimframework.com/docs/v4/)
- [Respect/Validation](https://respect-validation.readthedocs.io/)
- [Laminas InputFilter](https://docs.laminas.dev/laminas-inputfilter/)
- [Laminas Validator](https://docs.laminas.dev/laminas-validator/)
- [Mezzio Documentation](https://docs.mezzio.dev/)
- [CodeIgniter 4 Validation](https://codeigniter.com/user_guide/libraries/validation.html)
- [PHP-DI](https://php-di.org/)

### Project Internal
- [GENERATORS-COMMON.md](../GENERATORS-COMMON.md) - Template loop documentation
- [LARAVEL-SYMFONY-MAPPING.md](../openapi-generator-server-php-symfony-default/LARAVEL-SYMFONY-MAPPING.md)
- Existing templates: `openapi-generator-server-php-laravel/`, `openapi-generator-server-php-symfony/`
