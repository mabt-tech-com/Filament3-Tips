# Filament3-Tips




<br/><br/>

-----

<br/><br/>





## 1- Adding Tab Filters ( archive / all ) above your table :


<img src="https://i.imgur.com/jv4kMit.png" />

<img src="https://i.imgur.com/W8RIKi0.png" />


### 1.1. Enable Soft Deletes on Your Model
In `app/Models/Tags.php`:


```
use Illuminate\Database\Eloquent\SoftDeletes;

class Tags extends Model
{
    use SoftDeletes;
}
```


| ✅ Make sure your `tags` table has a `deleted_at` column. If not, create a migration:

```
php artisan make:migration add_deleted_at_to_tags_table
```

```
Schema::table('tags', function (Blueprint $table) {
    $table->softDeletes();
});
```

### 1.2. Add Tabs in Your Resource Page
In `app/Filament/Resources/TagsResource/Pages/ListTags.php`:

```
use Filament\Resources\Pages\ListRecords\Tab;
use App\Models\Tags;

class ListTags extends \Filament\Resources\Pages\ListRecords
{
    public function getTabs(): array
    {
        return [
            'all' => Tab::make(__('All'))
                ->badge(Tags::count()),

            'archived' => Tab::make(__('Archived'))
                ->badge(Tags::onlyTrashed()->count())
                ->modifyQueryUsing(fn ($query) => $query->onlyTrashed()),
        ];
    }
}

```


### 1.3. Make Sure Your Resource Supports Soft Deletes
In `TagsResource.php`, inside the `getEloquentQuery()` method, add this if needed:

```
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->withoutGlobalScopes([
        \Illuminate\Database\Eloquent\SoftDeletingScope::class,
    ]);
}
```

That’s it! ✅ You now have working tabs to filter "All" and "Archived" records using soft deletes.






<br/><br/>

-----

<br/><br/>




## 2- Customize the "Create" button in the header (change label + icon)

<img src="https://i.imgur.com/m2NSfLy.png" />


In your resource list page, e.g. `app/Filament/Resources/TagsResource/Pages/ListTags.php` :


```
use Filament\Actions;
use Filament\Support\Enums\IconPosition;

class ListTags extends \Filament\Resources\Pages\ListRecords
{
    protected function getHeaderActions(): array
    {
        return [
            Actions\CreateAction::make()
                ->label('New Tag') // Custom label
                ->icon('heroicon-o-plus-circle') // Custom icon
                ->iconPosition(IconPosition::Before), // Icon on the left
        ];
    }
}

```






<br/><br/>

-----

<br/><br/>



## 3- How to remove the delete button from The Edit Page : 

<img src="https://i.imgur.com/v0xRJjS.png" />

Open your Edit page class, e.g.
`app/Filament/Resources/TagsResource/Pages/EditTag.php` , 

Override the header‑actions to exclude Delete:

```
use Filament\Actions;
use Filament\Resources\Pages\EditRecord;

class EditTag extends EditRecord
{
    protected function getHeaderActions(): array
    {
        return [
                // REMOVE THE DeleteAction::make() from here : 
            // Actions\DeleteAction::make(),
            // no DeleteAction here!
        ];
    }
}

```




<br/><br/>

-----

<br/><br/>




## 4- Redirect to Index After Create/Edit in Filament : 

In your `CreateX.php` or `EditX.php` page, override the `getRedirectUrl()` method:


```
protected function getRedirectUrl(): string
{
    return static::getResource()::getUrl('index');
}
```


✅ Works for both create and edit pages.




<br/><br/>

-----

<br/><br/>



## 5- Reorderable Items Using position & Observer in Filament

<img src="https://i.imgur.com/RaliDLU.png" />

<img src="https://i.imgur.com/8vQRDCp.png" />



### 🧱 5.1. Migration Setup

Make sure your table has a `position` column defaulting to 0:


```
$table->integer('position')->default(0);

```

### ⚙️ 5.2. Auto-Assign position on Create

#### 5.2.a. Option 1: Use booted() in the model ✅

```
protected static function booted()
{
    static::creating(function (TrainingCategory $item) {
        $item->position = TrainingCategory::max('position') + 1;
    });
}

```


#### 5.2.b. Option 2: Hide the position field in the resource form + Auto-Assign in Resource


```

public static function form(Form $form): Form
    {
        return $form
            ->schema([ .....
                    // ADD this :     
                    Forms\Components\Hidden::make('position'),

```

Then in your resource’s `beforeCreate()`:

```
public static function beforeCreate($data): array
{
    $data['position'] = TrainingCategory::max('position') + 1;

    return $data;
}

```

### 👁️ 5.3. Keep Positions Consistent After Delete

When you delete an item, its position is removed — but the other items' positions stay the same, causing gaps (e.g., 1, 2, 4, 5…).
To fix this, use an observer that automatically re-indexes all remaining items after deletion.

#### 5.3.a. 🧠 Full Observer Logic (with explanation):

```
<?php

namespace App\Observers;

use App\Models\TrainingCategory;
use Illuminate\Support\Facades\DB;

class TrainingCategoryObserver
{
    // Assign the next available position when a new category is created
    public function creating(TrainingCategory $category)
    {
        $category->position = (int) TrainingCategory::max('position') + 1;
    }

    // After a category is deleted, re-order all remaining items to remove gaps
    public function deleted(TrainingCategory $category)
    {
        $orderColumn = 'position'; // the column we use to sort
        $keyName = $category->getKeyName(); // usually 'id'

        // Get all IDs ordered by current position
        $ordered = TrainingCategory::orderBy($orderColumn, 'asc')->pluck('id');

        // Build SQL CASE logic to reassign positions: 
        // e.g., case when id=3 then 1 when id=5 then 2 ...
        $cases = collect($ordered)
            ->map(fn ($key, int $index) => sprintf(
                'when %s = %s then %d',
                $keyName,
                DB::getPdo()->quote($key), // safely quote the ID for SQL
                $index + 1 // new position starts from 1
            ))
            ->implode(' ');

        // Perform one bulk update to apply the new positions
        TrainingCategory::whereIn('id', $ordered)->update([
            $orderColumn => DB::raw('case ' . $cases . ' end'),
        ]);
    }
}
```

#### 5.3.b. 🛠️ Register the Observer

In your `App\Providers\EventServiceProvider` :

```
use App\Models\TrainingCategory;
use App\Observers\TrainingCategoryObserver;

public function boot(): void
{
    TrainingCategory::observe(TrainingCategoryObserver::class);
}
```

#### 5.3.c. 🧪 Artisan Command to Create the Observer

Use this to quickly scaffold the observer:


```
php artisan make:observer TrainingCategoryObserver --model=TrainingCategory
```

This will place the file in `app/Observers/TrainingCategoryObserver.php` and prefill it with model methods like `created`, `deleted`, etc.




### 📦 5.4. Make Table Reorderable in Resource

In `TrainingCategoryResource.php` : 

```
->defaultSort('position')
->reorderable('position')
```




<br/><br/>

-----

<br/><br/>



## 6- Navigation Badge in Filament


<img src="https://i.imgur.com/6ke6lWd.png" />


#### 6.1. Place this code inside your Filament `Resource` class, e.g. `TrainingCategoryResource.php` :

```
public static function getNavigationBadge(): ?string
{
    return (string) static::getModel()::count();
}

public static function getNavigationBadgeColor(): ?string
{
    return 'purple'; // change color here
}

protected static ?string $navigationBadgeTooltip = 'Total number of trainings';

```


#### 6.2. 🎨 Available Colors (use in `getNavigationBadgeColor()` ):

```
primary, success, warning, danger, info, gray, secondary, blue, red, green, yellow, purple, pink, indigo, teal, orange, cyan, lime, amber, fuchsia, rose, sky, violet, stone, zinc, neutral, slate
```

✅ Just replace 'purple' with any of these.




<br/><br/>

-----

<br/><br/>



