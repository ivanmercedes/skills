---
name: "filament-resource-commands"
description: "Generate Filament resources for Filament v4 and v5 using the official Artisan generators instead of hand-crafting missing resource files. Use when the user needs to create, regenerate, or extend Filament resources, especially when the user asks for a resource that does not exist yet, wants CRUD scaffolding from a model, wants forms and tables generated from database columns with --generate, needs soft-delete support with --soft-deletes, or needs safe manual Filament v5 resource edits with the correct Forms-versus-Schemas imports."
---

# Filament Resource Commands

Create Filament resources with the official Artisan generator first. Do not scaffold a brand-new Filament resource one file at a time when the resource does not exist yet.

## Core rule

When a requested Filament resource does not exist, generate it with the official command before making manual refinements.

- Prefer `php artisan make:filament-resource Name` as the baseline.
- Prefer `php artisan make:filament-resource Name --generate` when the model and database columns already exist and the user wants the fastest CRUD scaffold.
- Prefer `php artisan make:filament-resource Name --soft-deletes` when the model uses soft deletes or the user explicitly wants trashed-record handling.
- Combine flags when that matches the task, for example `php artisan make:filament-resource Customer --generate --soft-deletes`.
- Do not manually create `Resource`, `Pages`, `Schemas`, or `Tables` classes from scratch if the generator can create them.

## First inspect the project

Before running the generator, inspect the Laravel and Filament project so the command matches the real app state.

1. Detect the Filament version from `composer.json`, `composer.lock`, or installed vendor packages.
2. Check whether the target Eloquent model already exists.
3. Check whether the resource already exists under the app's Filament resource namespace.
4. Check whether the model uses `SoftDeletes`.
5. Check whether the database schema already exists and is mature enough for `--generate` to be useful.
6. If the resource already exists, do not regenerate it blindly. Update only the missing pieces requested by the user.

## Command selection

Use this decision flow.

1. If the resource does not exist and the model exists with stable columns, run `php artisan make:filament-resource ModelName --generate`.
2. If the resource does not exist and the model uses soft deletes, add `--soft-deletes`.
3. If the resource does not exist but the schema is not ready or auto-generated fields would be noisy, run `php artisan make:filament-resource ModelName` and refine manually afterward.
4. If the user wants a simple modal resource, use `--simple`.
5. If the user also wants a View page, use `--view`.
6. If the model is not in `App\\Models`, use `--model-namespace=Custom\\Path\\Models`.

## Manual fallback for Filament v5

If the resource must be edited manually in Filament v5, follow the namespace split strictly.

- Import user-input fields from `Filament\Forms\Components`.
- Import layout and structure components from `Filament\Schemas\Components`.
- Import the form schema container from `Filament\Schemas\Schema`.
- Never assume every form-related class moved to `Filament\Schemas\Components` in v5.
- Do not import interactive fields like `Textarea`, `TextInput`, `Select`, or `Toggle` from `Filament\Schemas\Components`, because those classes stay under `Filament\Forms\Components`.

### Filament v5 input components

Use `Filament\Forms\Components` for data-entry fields such as:

```php
use Filament\Forms\Components\Checkbox;
use Filament\Forms\Components\DateTimePicker;
use Filament\Forms\Components\FileUpload;
use Filament\Forms\Components\Repeater;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\Toggle;
```

### Filament v5 layout components

Use `Filament\Schemas\Components` for layout-only components such as:

```php
use Filament\Schemas\Components\Grid;
use Filament\Schemas\Components\Group;
use Filament\Schemas\Components\Section;
use Filament\Schemas\Components\Split;
use Filament\Schemas\Components\Tabs;
use Filament\Schemas\Components\Tabs\Tab;
use Filament\Schemas\Components\Wizard;
```

### Filament v5 schema method

When editing a generated `...Form` class manually, prefer the v5 schema signature:

```php
use Filament\Schemas\Schema;

public static function configure(Schema $schema): Schema
{
    return $schema->components([
        // ...
    ]);
}
```

### Common import failure to avoid

If you see an error like `Class "Filament\Schemas\Components\Textarea" not found`, move `Textarea` back to `Filament\Forms\Components\Textarea`. Treat that pattern as a signal that an interactive field was imported from the wrong namespace.

## Working rules

- Never invent a missing Filament resource by manually creating every class one by one if the official generator should be used first.
- Never replace an existing working resource with a full regeneration unless the user explicitly asks for that risk.
- After generation, refine the generated resource, form schema, table schema, pages, filters, and actions to match the task.
- Preserve repository conventions and the existing Filament panel structure.
- If command execution is unavailable, explain that the correct next step is to run the generator, and avoid presenting a from-scratch manual scaffold as the preferred solution.
- If manual Filament v5 edits are required, keep the Forms-versus-Schemas namespace split correct before changing business logic.

## Typical commands

```bash
php artisan make:filament-resource Customer
php artisan make:filament-resource Customer --generate
php artisan make:filament-resource Customer --soft-deletes
php artisan make:filament-resource Customer --generate --soft-deletes
```

## After generating

1. Review the generated resource files before editing them.
2. Adjust generated form fields and table columns to match business rules.
3. Add validation, relationships, filters, actions, labels, navigation settings, and authorization as needed.
4. Run project-appropriate verification such as tests, static analysis, or a quick app check when possible.

## Reference

Read `references/filament-resource-generation.md` when you need the concise official command matrix, the Filament v5 import split, and version notes for Filament v4 and v5.
