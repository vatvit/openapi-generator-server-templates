# Laravel-Max Default Templates

## Note

**laravel-max** is a custom OpenAPI generator, not a built-in generator.

Unlike built-in generators (php-laravel, php-lumen, etc.), there are no "default" templates to extract from OpenAPI Generator.

## Template Location

Templates for laravel-max are located within the generator itself:

```
openapi-generator-generators/laravel-max/src/main/resources/laravel-max/
├── api-interface.mustache
├── controller.mustache
├── form-request.mustache
├── model.mustache
├── query-params.mustache
├── resource.mustache
├── routes.mustache
├── security-interface.mustache
├── security-validator.mustache
└── ...
```

## Why Templates Are in the Generator

Custom generators bundle their templates within the Java JAR. This is because:

1. The generator Java class controls which templates are used and how
2. Templates and generator logic are tightly coupled
3. No external template override is needed - modify templates in the generator source

## Customizing Templates

To customize templates:

1. Navigate to `openapi-generator-generators/laravel-max/`
2. Edit templates in `src/main/resources/laravel-max/`
3. Rebuild the generator: `make build`
4. Generate code: `make generate SPEC=... OUTPUT_DIR=...`

See `TEMPLATE-AUTHORING.md` in the parent directory for detailed guidance.
