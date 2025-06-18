# Filament3-Tips




<br/><br/>

-----

<br/><br/>





## 1- Adding Tab Filters ( archive / all ) above your table :


<img src="https://i.imgur.com/jv4kMit.png" />

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
            Actions\SaveAction::make(),
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

















