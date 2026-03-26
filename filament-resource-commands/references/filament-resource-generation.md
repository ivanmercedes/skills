# Filament resource generation reference

Use this reference when a Filament resource must be created and the generator should be preferred over manual scaffolding.

## Official commands

Filament 4.x and 5.x resources overview documents show these command patterns:

```bash
php artisan make:filament-resource Customer
php artisan make:filament-resource Customer --generate
php artisan make:filament-resource Customer --soft-deletes
```

## Practical guidance

- Use the plain command when you need the resource scaffold without schema-derived fields.
- Use `--generate` when the model and database columns already exist and you want Filament to generate forms and tables from the schema.
- Use `--soft-deletes` when the resource should support restore, force-delete, and trashed-record filtering.
- Combine `--generate` and `--soft-deletes` when both conditions apply.

## Filament v5 namespace split

In Filament v5, layout and schema components are separated from interactive form inputs.

- Import interactive data-entry fields from `Filament\Forms\Components`.
- Import layout and structure components from `Filament\Schemas\Components`.
- Import the schema container from `Filament\Schemas\Schema`.

### Inputs that stay in Forms

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

### Layout components that move to Schemas

```php
use Filament\Schemas\Components\Grid;
use Filament\Schemas\Components\Group;
use Filament\Schemas\Components\Section;
use Filament\Schemas\Components\Split;
use Filament\Schemas\Components\Tabs;
use Filament\Schemas\Components\Tabs\Tab;
use Filament\Schemas\Components\Wizard;
use Filament\Schemas\Schema;
```

## Common failure pattern

This error usually means an input component was imported from the wrong namespace:

```text
Class "Filament\Schemas\Components\Textarea" not found
```

Fix it by importing `Textarea` from `Filament\Forms\Components\Textarea`.

## Version note

- Filament 4.x official docs: `resources/overview`
- Filament 5.x official docs: `resources/overview`
- The older Filament 2.x admin resources getting started page shows the same command family and is still useful as historical confirmation, but prefer the 4.x and 5.x docs for current projects.
