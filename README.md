# Filament3-Tips




<br/><br/>

-----

<br/><br/>


## 1- Adding Tab Filters ( archive / all ) above your table :


<img src="https://i.imgur.com/jv4kMit.png" />

### 1. Enable Soft Deletes on Your Model
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

### 2. Add Tabs in Your Resource Page
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


### 3. Make Sure Your Resource Supports Soft Deletes
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



