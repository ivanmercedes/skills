---
name: filament-plugin
description: >
  Creates and manages modular Filament plugins in Laravel. Use this skill whenever the
  user asks to create a plugin, module, package, or feature for Filament. This includes:
  creating a new plugin in the plugins/ directory, generating its ServiceProvider,
  registering it as a local Composer path package, creating the Artisan Install command,
  and — if the plugin has migrations — configuring loadMigrationsFrom() so they can be
  published or physically copied into the main project. Trigger this skill when the user
  says things like "add a billing module", "create an inventory plugin", "I need a new
  Filament module for X", or any request that implies encapsulated functionality in Filament.
tags:
  - filamnent
---

# Filament Plugin Skill — Modular Laravel Architecture

## Concept

Each plugin lives in `plugins/{name}/` and is registered as a local Composer package via
a path repository in `composer.json`. The main project consumes it like any other Composer
package, keeping full independence and modularity.

## Required plugin structure

```
plugins/
└── {name}/
    ├── composer.json
    ├── src/
    │   ├── {Name}ServiceProvider.php
    │   ├── {Name}Plugin.php          <- Filament Plugin class
    │   └── Commands/
    │       └── Install{Name}Command.php
    ├── resources/
    │   ├── views/
    │   └── lang/
    ├── config/
    │   └── {name}.php
    └── database/
        └── migrations/
```

## Step 1 — Create `plugins/{name}/composer.json`

```json
{
    "name": "app/{name}",
    "description": "Modular {Name} plugin for Filament",
    "type": "library",
    "require": {
        "filament/filament": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "App\\Plugins\\{Name}\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "App\\Plugins\\{Name}\\{Name}ServiceProvider"
            ]
        }
    },
    "minimum-stability": "dev",
    "prefer-stable": true
}
```

## Step 2 — Register in the project root `composer.json`

Add the local path repository and require the package:

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "plugins/{name}",
            "options": {
                "symlink": true
            }
        }
    ],
    "require": {
        "app/{name}": "*"
    }
}
```

Then run:
```bash
composer require app/{name} --no-scripts
composer dump-autoload
```

> WARNING: If other path plugins already exist in repositories, only ADD the new entry —
> do NOT replace existing ones.

## Step 3 — ServiceProvider

File: `plugins/{name}/src/{Name}ServiceProvider.php`

```php
<?php

namespace App\Plugins\{Name};

use Illuminate\Support\ServiceProvider;

class {Name}ServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->mergeConfigFrom(
            __DIR__ . '/../config/{name}.php',
            '{name}'
        );
    }

    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__ . '/../resources/views', '{name}');
        $this->loadTranslationsFrom(__DIR__ . '/../resources/lang', '{name}');

        // Migrations are loaded directly from the plugin so `migrate` works
        // out of the box. The Install command copies them physically (Step 5).
        $this->loadMigrationsFrom(__DIR__ . '/../database/migrations');

        if ($this->app->runningInConsole()) {
            $this->publishes([
                __DIR__ . '/../database/migrations' => database_path('migrations'),
            ], '{name}-migrations');

            $this->publishes([
                __DIR__ . '/../config/{name}.php' => config_path('{name}.php'),
            ], '{name}-config');

            $this->publishes([
                __DIR__ . '/../resources/views' => resource_path('views/vendor/{name}'),
            ], '{name}-views');

            $this->commands([
                \App\Plugins\{Name}\Commands\Install{Name}Command::class,
            ]);
        }
    }
}
```

## Step 4 — Filament Plugin Class

File: `plugins/{name}/src/{Name}Plugin.php`

```php
<?php

namespace App\Plugins\{Name};

use Filament\Contracts\Plugin;
use Filament\Panel;

class {Name}Plugin implements Plugin
{
    public function getId(): string
    {
        return '{name}';
    }

    public function register(Panel $panel): void
    {
        // Register resources, pages, widgets, etc.
        // Example:
        // $panel->resources([
        //     \App\Plugins\{Name}\Resources\{Name}Resource::class,
        // ]);
    }

    public function boot(Panel $panel): void
    {
        //
    }

    public static function make(): static
    {
        return app(static::class);
    }
}
```

**Important**: Remind the user to register the plugin in their Panel Provider:
```php
// app/Providers/Filament/AdminPanelProvider.php
->plugins([
    \App\Plugins\{Name}\{Name}Plugin::make(),
])
```

## Step 5 — Install Command (with migration copy)

File: `plugins/{name}/src/Commands/Install{Name}Command.php`

```php
<?php

namespace App\Plugins\{Name}\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\File;

class Install{Name}Command extends Command
{
    protected $signature   = '{name}:install {--force : Overwrite existing files}';
    protected $description = 'Install the {Name} plugin and publish its assets';

    public function handle(): int
    {
        $this->info('Installing {Name} plugin...');

        // 1. Publish config
        $this->callSilently('vendor:publish', [
            '--tag'   => '{name}-config',
            '--force' => $this->option('force'),
        ]);
        $this->line('  [OK] Config published');

        // 2. Copy migrations into the project
        $this->publishMigrations();

        // 3. Publish views (optional)
        if ($this->confirm('Would you like to publish the views for customization?', false)) {
            $this->callSilently('vendor:publish', [
                '--tag'   => '{name}-views',
                '--force' => $this->option('force'),
            ]);
            $this->line('  [OK] Views published');
        }

        $this->newLine();
        $this->info('{Name} plugin installed successfully.');
        $this->line('  -> Remember to add <comment>{Name}Plugin::make()</comment> to your Panel Provider.');
        $this->line('  -> Run <comment>php artisan migrate</comment> to apply the migrations.');

        return self::SUCCESS;
    }

    protected function publishMigrations(): void
    {
        $source      = __DIR__ . '/../../database/migrations';
        $destination = database_path('migrations');

        if (! File::isDirectory($source)) {
            $this->line('  [i] No migrations found in this plugin');
            return;
        }

        $files   = File::files($source);
        $copied  = 0;
        $skipped = 0;

        foreach ($files as $file) {
            $target = $destination . '/' . $file->getFilename();

            if (File::exists($target) && ! $this->option('force')) {
                $skipped++;
                continue;
            }

            File::copy($file->getPathname(), $target);
            $copied++;
        }

        if ($copied > 0) {
            $this->line("  [OK] {$copied} migration(s) copied to database/migrations");
        }
        if ($skipped > 0) {
            $this->line("  [i] {$skipped} migration(s) skipped (already exist — use --force to overwrite)");
        }
    }
}
```

## Step 6 — Minimal supporting files

### `plugins/{name}/config/{name}.php`
```php
<?php

return [
    'enabled' => true,
];
```

### `plugins/{name}/database/migrations/` (if applicable)
Name migrations with a timestamp + description:
```
2024_01_01_000000_create_{name}_table.php
```

## Naming conventions

| Token      | Example (name = "invoicing")      |
|------------|-----------------------------------|
| `{name}`   | `invoicing`                       |
| `{Name}`   | `Invoicing`                       |
| namespace  | `App\Plugins\Invoicing`           |
| package    | `app/invoicing`                   |
| directory  | `plugins/invoicing/`              |
| command    | `invoicing:install`               |

For compound names (e.g. "purchase order"):
- `{name}` -> `purchase-order` (kebab-case for Composer and directory)
- `{Name}` -> `PurchaseOrder` (PascalCase for classes)
- namespace -> `App\Plugins\PurchaseOrder`

## Delivery checklist

After creating a plugin, always show this summary to the user:

```
[OK] Plugin {Name} created at plugins/{name}/
[OK] Registered in composer.json (path repository)
[OK] ServiceProvider with loadMigrationsFrom()
[OK] Install command: php artisan {name}:install

Next steps:
   1. composer require app/{name}
   2. php artisan {name}:install
   3. Add {Name}Plugin::make() to your Panel Provider
   4. php artisan migrate
```

## Additional reference

For complex plugins with Filament Resources, Pages, and Widgets, see:
`references/filament-components.md`

For advanced migration handling and versioning, see:
`references/migrations-advanced.md`
