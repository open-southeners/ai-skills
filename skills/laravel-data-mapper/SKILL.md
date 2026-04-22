---
name: laravel-data-mapper
description: Use when authoring or reviewing Laravel code that uses the open-southeners/laravel-data-mapper package (composer name open-southeners/laravel-dto) — DataTransferObject classes, PropertiesMapper, TypeGenerator, mapping attributes, or any composer.json with this dependency. Covers defining DTOs, mapping request/array data into them, and generating TypeScript types from them.
---

# laravel-data-mapper

Helps build and review DTOs in Laravel using `open-southeners/laravel-data-mapper` (Composer package: `open-southeners/laravel-dto`).

## Source of truth (in this order)

> ⚠️ **Doc maturity note:** the GitBook docs for this package are still being filled out. When in doubt, the package source and tests are the authority — not the docs.

1. **The package source** at `vendor/open-southeners/laravel-dto/src/` (`DataTransferObject.php`, `PropertiesMapper.php`, `TypeGenerator.php`, `Attributes/`, `Commands/`, `Contracts/`) and tests at `vendor/open-southeners/laravel-dto/tests/`. **This is the authority.**
2. **GitBook MCP** for the `laravel-data-mapper` docs space, if configured.
3. **`WebFetch`** against `https://docs.opensoutheners.com/laravel-data-mapper/` (key pages: `creating-dtos`, `usage`, `typescript`).

For any class signature, mapping attribute, or behavior question: read the PHP source first.

## Mental model

The package gives you three things:

1. **`DataTransferObject`** — a base class for typed, immutable data containers.
2. **`PropertiesMapper`** — the engine that takes an input array (e.g. a request payload) and maps it into DTO properties, honoring attributes on those properties.
3. **`TypeGenerator`** — emits TypeScript type definitions from PHP DTO classes (driven by an Artisan command in `Commands/`).

Mapping behavior is driven by **attributes** in `src/Attributes/` (e.g. casting, source key, default values). Always check that directory before assuming an attribute exists or behaves a certain way.

## Author flow

When asked to add or change a DTO:

1. **Read `Attributes/`** in the installed package to see exactly which attributes are available and what they do. Don't assume Laravel-Data or spatie/laravel-data semantics — this is a different package.
2. **Subclass `DataTransferObject`** with typed properties. Use PHP property attributes for mapping behavior (source key remapping, casts, defaults).
3. **Construct via the documented entry point.** Find it by reading `DataTransferObject.php` — typically a static factory that delegates to `PropertiesMapper`. Don't `new` DTOs directly unless the source confirms that's the intended path.
4. **For Form Request integration**, check whether the package ships a trait/contract for hydrating from a `FormRequest` — if so, use it; if not, hydrate explicitly in the controller.
5. **TypeScript generation**: if the project already runs `TypeGenerator`, register the new DTO so it gets emitted. Look in `Commands/` for the Artisan command name and any config it reads.

## Review flow

When asked to review code that uses this package, check:

- **Direct `new DTO(...)` calls** that bypass `PropertiesMapper` — they skip attribute-driven mapping (casts, key remaps, defaults) and produce subtly broken DTOs.
- **Untyped or loosely-typed properties** on DTOs — defeats the purpose; the mapper relies on types for casting.
- **Manual array munging** before passing to the mapper — often unnecessary; an attribute (source key remap, cast) usually exists for it. Check `Attributes/`.
- **DTOs treated as mutable** — they're meant to be immutable data carriers. Setter-style code is a smell.
- **TypeScript drift**: if `TypeGenerator` runs in CI, new/changed DTOs without a regenerated `.ts` artifact will diverge.
- **Missing tests**: DTO mapping logic is easy to test with array fixtures; absence of tests around custom attributes/casts is a real gap.

## Common gotchas

- The Composer name (`open-southeners/laravel-dto`) and the repository name (`laravel-data-mapper`) differ. Both refer to the same package — don't get confused when reading `composer.json` vs. docs URLs.
- Because the docs are incomplete, **never recommend an attribute or method without grepping the package source first** to confirm it exists in the installed version.
- `TypeGenerator` reflects on PHP types — union types, intersection types, and generics may have edge cases; check tests under `tests/` for the typescript generator before promising a given PHP→TS mapping.

## When to defer to the user

If the source has a feature that isn't documented at all, surface that to the user before recommending it ("the source supports X but it isn't documented — confirm this is part of the public API before relying on it").
