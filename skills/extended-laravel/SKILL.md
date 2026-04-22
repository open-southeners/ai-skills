---
name: extended-laravel
description: Use when working on a Laravel project that has open-southeners/extended-laravel installed, before writing helper functions, custom casts, validation rules, console commands, middleware, or facades from scratch. The package is a grab-bag of Laravel utilities; this skill helps discover what's already there so you don't reinvent it. Also use when reviewing code that imports from open-southeners/extended-laravel.
---

# extended-laravel

A discovery/reference skill for `open-southeners/extended-laravel`. The package is a collection of Laravel utilities (helpers, casts, validation rules, console commands, middleware, facades, events/listeners). There isn't a single "author flow" — instead, this skill helps you check whether a needed utility already exists before writing a new one.

## Source of truth (in this order)

1. **GitBook MCP** for the `extended-laravel` docs space, if configured.
2. **`WebFetch`** against `https://docs.opensoutheners.com/extended-laravel/` and its sub-pages (`commands`, `facades`, `function-helpers`, `middleware`).
3. **The package source** at `vendor/open-southeners/extended-laravel/src/`, organized by area:
   - `Casts/` — Eloquent casts
   - `Console/` — Artisan commands
   - `Events/` and `Listeners/` — event-driven utilities
   - `Helpers.php` and `Helpers/` — global helper functions
   - `Http/` — middleware, request macros
   - `Support/` — misc support classes
   - `Validation/` — custom validation rules
   - `Config/` — config helpers
   - `AboutCommandIntegration.php` — integrates with `php artisan about`

## How to use this skill

When the user (or you) is about to:

- **Write a helper function** → grep `vendor/open-southeners/extended-laravel/src/Helpers.php` and `Helpers/` first.
- **Add a custom Eloquent cast** → check `src/Casts/`.
- **Write a validation rule** → check `src/Validation/`.
- **Add an Artisan command for a routine task** → check `src/Console/`.
- **Add middleware (auth, request shaping, response formatting)** → check `src/Http/`.
- **Reach for a Laravel facade that doesn't exist in core** → check `src/Support/` and the `facades.md` doc page.

If a fitting utility exists, use it and tell the user where it came from. If not, proceed to write the new code as normal.

## Review flow

When reviewing code that imports from `open-southeners/extended-laravel`:

- Confirm the imported symbol still exists in the installed version (the package evolves; old docs/snippets may reference renamed/removed APIs).
- Flag local re-implementations of utilities the package already provides (the whole point of having the package installed is to use it).
- Check that custom casts/validation rules from the package are registered correctly in the consuming app where required.

## Scope note

This skill intentionally does **not** cover authoring contributions to the `extended-laravel` package itself — only consumption from a downstream Laravel app. If the user is editing the package's own source, the local repo's conventions and tests are the authority; this skill is the wrong tool.
