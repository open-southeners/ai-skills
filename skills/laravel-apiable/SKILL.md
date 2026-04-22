---
name: laravel-apiable
description: Use when authoring or reviewing Laravel code that uses the open-southeners/laravel-apiable package — JSON:API responses, JsonApiResponse, JsonApiResource/Collection, Apiable::toJsonApi(), Allowed filters/sorts/fields/includes, jsonApiPaginate(), or any composer.json with "open-southeners/laravel-apiable". Covers building filterable, sortable, and includable JSON:API endpoints, applying the package's pipeline correctly, and reviewing existing code for the package's idioms.
---

# laravel-apiable

Helps build and review JSON:API endpoints in Laravel using `open-southeners/laravel-apiable`.

## Source of truth (in this order)

1. **GitBook MCP** for the `laravel-apiable` docs space, if the user has it configured. Query it for any concept, class, or config you're unsure about.
2. **`WebFetch`** against `https://docs.opensoutheners.com/laravel-apiable/` and its sub-pages (e.g. `/getting-started`, `/requests`, `/responses`, `/advanced`) when MCP isn't available.
3. **The package source** at `vendor/open-southeners/laravel-apiable/src/` and its tests at `vendor/open-southeners/laravel-apiable/tests/`. When docs and code disagree, **trust the code and tests**.

Always retrieve before recommending — don't lean on pre-trained knowledge for class names, method signatures, or config keys.

## Mental model

Every JSON:API response flows through a 5-stage pipeline:

1. `Apiable::toJsonApi($resource)` — entry point, dispatches by resource type
2. Pipeline stages applied to the query: `ApplyFulltextSearchToQuery` (Scout), `ApplyFiltersToQuery`, `ApplyIncludesToQuery`, `ApplyFieldsToQuery` (sparse fieldsets), `ApplySortsToQuery`
3. Pagination via the `jsonApiPaginate()` Builder macro (length-aware / simple / cursor — controlled by `config('apiable.responses.pagination.type')`, overridable per-response with `simplePaginating()` / `cursorPaginating()`)
4. Resource serialization via `JsonApiResource` / `JsonApiCollection` (with `RelationshipsWithIncludes`, `CollectsWithIncludes` traits)
5. Post-processing (appends, included deduplication) via `IteratesResultsAfterQuery`

Filters, sorts, fields, and includes are declared via `Allowed*` classes attached to the response, not free-form on the request.

Models map to JSON:API types via `config('apiable.resource_type_map')`.

## Author flow

When asked to add or change an endpoint:

1. **Confirm Accept header behavior expected.** The package negotiates on `application/vnd.api+json` and falls back to `config('apiable.responses.formatting.type')` (e.g. Inertia). It throws 406 for unsupported Accepts.
2. **Identify the model and its resource type.** Make sure it's registered in `config/apiable.php` under `resource_type_map`.
3. **Build the response** with `Apiable::toJsonApi(Model::query())` (or a resource class) and chain `Allowed*` declarations:
   - `allowFilter(...)` with the right operator (`SIMILAR`, `EQUAL`, etc. — default at `config('apiable.requests.filters.default_operator')`)
   - `allowSort(...)` (relationship sorting auto-JOINs — check `Builder::hasJoin()` to avoid duplicates)
   - `allowInclude(...)` — respect `responses.max_include_depth` (default 3)
   - `allowFields(...)` for sparse fieldsets
4. **Pick the pagination strategy** intentionally. Length-aware does a `COUNT` query — avoid on very large tables; prefer `simplePaginating()` or `cursorPaginating()`.
5. **Check tests in the package** (`vendor/.../tests/Http/JsonApiResponseTest.php` and siblings) for the exact API shape you're using. Don't guess method names.

## Review flow

When asked to review code that uses this package, check:

- **Pipeline misuse**: free-form `where()` chains layered after `allowFilter` declarations that the user expected to be driven by the request — they won't be filtered by the request unless declared.
- **Resource type mapping** missing from `apiable.resource_type_map` — silently produces wrong `type` values.
- **Pagination strategy mismatch**: length-aware on a giant table; cursor pagination used where stable offsets matter.
- **Include depth**: deeply nested `allowInclude` chains exceeding `max_include_depth` — they get truncated, often unnoticed.
- **N+1 from includes** that aren't covered by `ApplyIncludesToQuery`'s eager loading (custom resource accessors that hit DB).
- **Per-response config overrides** done with `config(['apiable.x' => y])` — make sure they aren't leaking across requests (the package's convenience methods scope this correctly; ad-hoc calls may not).
- **Custom `JsonApiResource` subclasses** that override `toArray()` directly instead of leveraging the trait pipeline — usually a smell.
- **Tests**: feature tests should hit `application/vnd.api+json` accept and assert structure with `assertJsonApi*` helpers from `src/Testing/` if they exist — check the testing namespace.

## Common gotchas

- `Builder.php` uses Laravel's mixin pattern (closures returning `$this` context). Don't try to override these statically; extend via the same mixin pattern.
- Hash-set deduplication in `RelationshipsWithIncludes` / `CollectsWithIncludes` is performance-critical — don't replace with `Collection::unique()` "for clarity."
- `viewable` query scoping (`config('apiable.responses.viewable')`) silently filters records — surface this when reviewing authorization-adjacent code.

## When to defer to the user

If the docs and the code disagree on a non-trivial point, surface both and ask the user which is current. The package is actively maintained, and recent perf optimizations changed internals without always landing in docs immediately.
