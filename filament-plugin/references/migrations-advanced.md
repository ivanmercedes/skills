# Advanced Migrations in Plugins

## Migrations with plugin prefix (recommended to avoid conflicts)

Name migrations with the plugin prefix to avoid collisions with other plugins:

```
database/migrations/
├── 2024_01_01_000001_create_invoicing_invoices_table.php
├── 2024_01_01_000002_create_invoicing_items_table.php
└── 2024_01_01_000003_create_invoicing_payments_table.php
```

## Example migration with good practices

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{name}s', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->boolean('active')->default(true);
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{name}s');
    }
};
```

## Strategy: loadMigrationsFrom vs. physical copy

### `loadMigrationsFrom()` (automatic, no copy)
- Migrations run directly from the plugin when the user calls `php artisan migrate`
- The file does NOT appear in the project's `database/migrations/`
- Useful when you do NOT want users to modify the migrations

### Physical copy via the Install command (recommended for modular projects)
- Copies files to the project's `database/migrations/`
- The user can review and modify migrations before running them
- More transparent and controllable
- **This is the default approach used by this skill**

### Hybrid (both)
The ServiceProvider uses `loadMigrationsFrom()` so `migrate` works without installation.
The Install command copies files physically for projects that prefer them in the repo.

## Install command with migration verification

```php
protected function publishMigrations(): void
{
    $source      = __DIR__ . '/../../database/migrations';
    $destination = database_path('migrations');
    $files       = \Illuminate\Support\Facades\File::files($source);

    if (empty($files)) {
        $this->line('  [i] No migrations in this plugin');
        return;
    }

    $this->line('  Migrations found: ' . count($files));

    foreach ($files as $file) {
        $target = $destination . '/' . $file->getFilename();

        if (\Illuminate\Support\Facades\File::exists($target) && ! $this->option('force')) {
            $this->line("  [skip] Already exists: {$file->getFilename()}");
            continue;
        }

        \Illuminate\Support\Facades\File::copy($file->getPathname(), $target);
        $this->line("  [OK] Copied: {$file->getFilename()}");
    }
}
```
