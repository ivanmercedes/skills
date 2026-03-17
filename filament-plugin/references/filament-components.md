# Filament Components Inside Plugins

## Resource inside a plugin

```php
// plugins/{name}/src/Resources/{Name}Resource.php
<?php

namespace App\Plugins\{Name}\Resources;

use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Forms;
use Filament\Tables\Table;
use Filament\Forms\Form;

class {Name}Resource extends Resource
{
    // protected static ?string $model = \App\Plugins\{Name}\Models\{Name}::class;
    protected static ?string $navigationIcon  = 'heroicon-o-rectangle-stack';
    protected static ?string $navigationGroup = '{Name}';

    public static function form(Form $form): Form
    {
        return $form->schema([
            //
        ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                //
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ]);
    }

    public static function getPages(): array
    {
        return [
            'index'  => Pages\List{Name}::route('/'),
            'create' => Pages\Create{Name}::route('/create'),
            'edit'   => Pages\Edit{Name}::route('/{record}/edit'),
        ];
    }
}
```

Register in the Plugin class:
```php
public function register(Panel $panel): void
{
    $panel->resources([
        \App\Plugins\{Name}\Resources\{Name}Resource::class,
    ]);
}
```

## Widget inside a plugin

```php
// plugins/{name}/src/Widgets/{Name}StatsWidget.php
<?php

namespace App\Plugins\{Name}\Widgets;

use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class {Name}StatsWidget extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total', 0),
        ];
    }
}
```

Register in the Plugin class:
```php
public function register(Panel $panel): void
{
    $panel->widgets([
        \App\Plugins\{Name}\Widgets\{Name}StatsWidget::class,
    ]);
}
```

## Custom Page inside a plugin

```php
// plugins/{name}/src/Pages/{Name}Dashboard.php
<?php

namespace App\Plugins\{Name}\Pages;

use Filament\Pages\Page;

class {Name}Dashboard extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-home';
    protected static string  $view = '{name}::pages.dashboard';
}
```

Register in the Plugin class:
```php
public function register(Panel $panel): void
{
    $panel->pages([
        \App\Plugins\{Name}\Pages\{Name}Dashboard::class,
    ]);
}
```

## Model inside a plugin

```php
// plugins/{name}/src/Models/{Name}.php
<?php

namespace App\Plugins\{Name}\Models;

use Illuminate\Database\Eloquent\Model;

class {Name} extends Model
{
    protected $table    = '{name}s'; // adjust as needed
    protected $fillable = [];
}
```
