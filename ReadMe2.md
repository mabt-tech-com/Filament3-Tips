### Comprehensive Tutorial: Reusable Tabs for Any Filament Resource


<img src="https://i.imgur.com/qKAuX9W.png" alt="picture" >

#### Step 1: Create the Tab Helper & Component (One-Time Setup)
**File: `app/Filament/Helpers/TabHelper.php`**
```php
<?php

namespace App\Filament\Helpers;

class TabHelper
{
    public static function makeTabs(array $config): array
    {
        return array_map(function ($tab) {
            return [
                'key' => $tab['key'],
                'label' => $tab['label'],
                'badge' => $tab['badge'] ?? null,
                'badgeColor' => $tab['badgeColor'] ?? 'primary',
                'icon' => $tab['icon'] ?? 'heroicon-o-document-text',
            ];
        }, $config);
    }
}
```

**File: `resources/views/components/filament/reusable-tabs.blade.php`**
```blade
@props([
    'tabs' => [],
    'currentTab' => 'all',
    'wireModel' => 'statusTab',
    'style' => 'margin-bottom: -24px; border-bottom-left-radius: 5; border-bottom-right-radius: 5;',
])

<x-filament::tabs class="w-full" :style="$style">
    @foreach ($tabs as $tab)
        <x-filament::tabs.item
            class="whitespace-nowrap"
            wire:click="$set('{{ $wireModel }}', '{{ $tab['key'] }}')"
            :active="$currentTab === $tab['key']"
            :badge="$tab['badge']"
            :badge-color="$tab['badgeColor']"
            :icon="$tab['icon']"
        >
            {{ $tab['label'] }}
        </x-filament::tabs.item>
    @endforeach
</x-filament::tabs>
```

#### Step 2: For Any New Resource (e.g., ProductResource)

**1. Create the Resource**
```bash
php artisan make:filament-resource Product
```

**2. Create Custom List Page**
```bash
php artisan make:filament-page ListProducts --resource --type=ListRecords
```

**3. Configure List Page Logic**
**File: `app/Filament/Resources/ProductResource/Pages/ListProducts.php`**
```php
<?php

namespace App\Filament\Resources\ProductResource\Pages;

use App\Filament\Helpers\TabHelper;
use App\Filament\Resources\ProductResource;
use App\Models\Product;
use Filament\Resources\Pages\ListRecords;
use Illuminate\Database\Eloquent\Builder;

class ListProducts extends ListRecords
{
    protected static string $resource = ProductResource::class;
    protected static string $view = 'filament.resources.product-resource.pages.list-products';
    
    public string $statusTab = 'all';
    public array $staticTabs = [];

    protected array $queryString = [
        'statusTab' => ['except' => 'all'],
    ];

    public function mount(): void
    {
        parent::mount();
        
        $this->staticTabs = TabHelper::makeTabs([
            [
                'key' => 'all',
                'label' => 'All Products',
                'badge' => Product::count(),
            ],
            [
                'key' => 'in_stock',
                'label' => 'In Stock',
                'badge' => Product::where('stock', '>', 0)->count(),
                'badgeColor' => 'success',
                'icon' => 'heroicon-o-check-circle',
            ],
            [
                'key' => 'out_of_stock',
                'label' => 'Out of Stock',
                'badge' => Product::where('stock', 0)->count(),
                'badgeColor' => 'danger',
                'icon' => 'heroicon-o-x-circle',
            ],
            // Add more tabs as needed
        ]);
    }

    protected function getTableQuery(): Builder
    {
        $query = parent::getTableQuery();

        // Apply tab-specific filters
        match ($this->statusTab) {
            'in_stock' => $query->where('stock', '>', 0),
            'out_of_stock' => $query->where('stock', 0),
            default => $query, // 'all' tab
        };

        return $query;
    }
}
```

**4. Create the Blade View from Template**
1. Find the default template:
   `vendor/filament/filament/resources/views/resources/pages/list-records.blade.php`

2. Copy its contents

3. Create new file:
   `resources/views/filament/resources/product-resource/pages/list-products.blade.php`

4. Paste template content and add tabs component:
```blade
<x-filament-panels::page
    @class([
        'fi-resource-list-records-page',
        'fi-resource-' . str_replace('/', '-', $this->getResource()::getSlug()),
    ])
>
    <div class="flex flex-col gap-y-6">
        <x-filament-panels::resources.tabs />

        {{ \Filament\Support\Facades\FilamentView::renderHook(
             \Filament\View\PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_BEFORE,
             scopes: $this->getRenderHookScopes()
        ) }}

        {{-- ADD REUSABLE TABS HERE --}}
        <x-filament.reusable-tabs
            :tabs="$staticTabs"
            :currentTab="$statusTab"
            wireModel="statusTab"
        />

        {{ $this->table }}

        {{ \Filament\Support\Facades\FilamentView::renderHook(
             \Filament\View\PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_AFTER,
             scopes: $this->getRenderHookScopes()
        ) }}
    </div>
</x-filament-panels::page>
```

**5. Update Resource Registration**
**File: `app/Filament/Resources/ProductResource.php`**
```php
class ProductResource extends Resource
{
    // ...

    public static function getPages(): array
    {
        return [
            'index' => Pages\ListProducts::route('/'),
            'create' => Pages\CreateProduct::route('/create'),
            'edit' => Pages\EditProduct::route('/{record}/edit'),
        ];
    }
}
```

### For Any Future Resource (General Pattern)

1. **Create Resource**
   ```bash
   php artisan make:filament-resource YourResourceName
   ```

2. **Create Custom List Page**
   ```bash
   php artisan make:filament-page ListYourResources \
     --resource \
     --type=ListRecords
   ```

3. **Configure List Page**
    - Add properties: `$statusTab` and `$staticTabs`
    - Implement `mount()` with tab configuration
    - Add filtering in `getTableQuery()`

4. **Create Blade View**
    - Copy content from:
      `vendor/filament/filament/resources/views/resources/pages/list-records.blade.php`
    - Create file at:
      `resources/views/filament/resources/your-resource-name-resource/pages/list-your-resources.blade.php`
    - Insert reusable tabs component before `{{ $this->table }}`

5. **Update Resource Class**
    - Point 'index' route to your custom list page

### Key Files Structure
```
app/
├── Filament/
│   ├── Helpers/
│   │   └── TabHelper.php
│   └── Resources/
│       ├── ProductResource/
│       │   ├── Pages/
│       │   │   ├── ListProducts.php
│       │   │   ├── CreateProduct.php
│       │   │   └── EditProduct.php
│       │   └── ProductResource.php
│       └── YourResource/
│           └── ... (same pattern)
resources/
├── views/
│   ├── components/
│   │   └── filament/
│   │       └── reusable-tabs.blade.php
│   └── filament/
│       └── resources/
│           ├── product-resource/
│           │   └── pages/
│           │       └── list-products.blade.php
│           └── your-resource/
│               └── pages/
│                   └── list-your-resources.blade.php
```

### Customization Options

1. **Tab Configuration**:
   ```php
   [
       'key' => 'unique_key',      // Required
       'label' => 'Display Name',  // Required
       'badge' => 42,              // Optional count
       'badgeColor' => 'warning',  // Optional (primary, success, danger, warning, info)
       'icon' => 'heroicon-o-star' // Optional Heroicon
   ]
   ```

2. **Query Filtering**:
   ```php
   protected function getTableQuery(): Builder
   {
       $query = parent::getTableQuery();
       
       match ($this->statusTab) {
           'tab1' => $query->where('column', 'value'),
           'tab2' => $query->whereRelation('relation', 'field', 'condition'),
           default => $query,
       };
       
       return $query;
   }
   ```

3. **Component Props**:
   ```blade
   <x-filament.reusable-tabs
       :tabs="$staticTabs"
       :currentTab="$statusTab"
       wireModel="customTabState"  <!-- Change Livewire property name -->
       style="margin-bottom: 0;"   <!-- Custom styles -->
   />
   ```

### Troubleshooting Tips

1. **Blank Page**:
    - Verify blade file path matches `$view` property
    - Check for syntax errors in blade file

2. **Tabs Not Appearing**:
    - Ensure `$staticTabs` is populated in `mount()`
    - Verify component path: `components/filament/reusable-tabs.blade.php`

3. **Filter Not Working**:
    - Check `getTableQuery()` logic matches tab keys
    - Add `dd($this->statusTab)` to verify current tab

4. **Badge Counts Wrong**:
    - Ensure counts are calculated correctly in `mount()`
    - Use model scopes for better readability:
      ```php
      'badge' => Product::inStock()->count(),
      ```

This pattern works for any Filament resource and provides a consistent way to add tabbed navigation. The key steps are:
1. Create custom list page
2. Configure tabs in PHP
3. Create blade view from template + add component
4. Implement query filtering

All future resources can follow this exact pattern - just replace "Product" with your model name and customize the tab configuration.
