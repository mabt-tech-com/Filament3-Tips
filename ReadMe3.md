# Advanced Pagination for Filament Tables


<img src="https://i.imgur.com/BJ7rZfD.png" />


Here's a comprehensive `README.md` tutorial for implementing advanced pagination in Filament, including solutions for common issues:

```markdown
# Advanced Pagination for Filament Tables

This tutorial shows how to enhance Filament's default pagination with:
- First/Last page links
- Next/Previous arrows
- "Go to page" input
- Result count display
- Page number navigation

## Step 1: Publish Pagination Views

Run this command to publish Filament's pagination components:

```bash
php artisan vendor:publish --tag=filament-pagination-views
```

**Troubleshooting Tip**: If the `pagination.blade.php` file isn't created automatically:
1. Create the directory path manually:
   ```bash
   mkdir -p resources/views/vendor/filament/components/table
   ```
2. Create the file:
   ```bash
   touch resources/views/vendor/filament/components/table/pagination.blade.php
   ```

## Step 2: Customize the Pagination View

Edit `resources/views/vendor/filament/components/table/pagination.blade.php` with this code:

```blade
@if ($paginator->hasPages())
    <div class="flex flex-col md:flex-row items-center justify-between gap-4 py-3">
        {{-- Result Count --}}
        <div class="text-sm text-gray-700 dark:text-gray-300">
            Showing
            <span class="font-semibold">{{ $paginator->firstItem() }}</span>
            to
            <span class="font-semibold">{{ $paginator->lastItem() }}</span>
            of
            <span class="font-semibold">{{ $paginator->total() }}</span>
            results
        </div>

        <div class="flex items-center space-x-1">
            {{-- First Page Link --}}
            <button 
                wire:click="gotoPage(1)" 
                @disabled($paginator->onFirstPage())
                class="px-2 py-1 rounded-md text-gray-700 dark:text-gray-300 disabled:opacity-50"
                aria-label="First Page"
            >
                &laquo;
            </button>

            {{-- Previous Page --}}
            <button 
                wire:click="previousPage" 
                @disabled($paginator->onFirstPage())
                class="px-2 py-1 rounded-md text-gray-700 dark:text-gray-300 disabled:opacity-50"
                aria-label="Previous Page"
            >
                &lsaquo;
            </button>

            {{-- Page Numbers --}}
            @foreach ($elements as $element)
                @if (is_string($element))
                    <span class="px-2 py-1">{{ $element }}</span>
                @endif
                
                @if (is_array($element))
                    @foreach ($element as $page => $url)
                        <button
                            wire:click="gotoPage({{ $page }})"
                            @class([
                                'px-3 py-1 rounded-md',
                                'bg-primary-500 text-white font-medium' => $page == $paginator->currentPage(),
                                'text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700' => $page != $paginator->currentPage(),
                            ])
                            aria-label="Page {{ $page }}"
                        >
                            {{ $page }}
                        </button>
                    @endforeach
                @endif
            @endforeach

            {{-- Next Page --}}
            <button 
                wire:click="nextPage" 
                @disabled(!$paginator->hasMorePages())
                class="px-2 py-1 rounded-md text-gray-700 dark:text-gray-300 disabled:opacity-50"
                aria-label="Next Page"
            >
                &rsaquo;
            </button>

            {{-- Last Page Link --}}
            <button 
                wire:click="gotoPage({{ $paginator->lastPage() }})" 
                @disabled(!$paginator->hasMorePages())
                class="px-2 py-1 rounded-md text-gray-700 dark:text-gray-300 disabled:opacity-50"
                aria-label="Last Page"
            >
                &raquo;
            </button>

            {{-- Go to Page Input --}}
            <div class="ml-4 flex items-center">
                <span class="text-sm text-gray-700 dark:text-gray-300 mr-2">Go to:</span>
                <form 
                    wire:submit.prevent="gotoPage($event.target.page.value)"
                    class="flex"
                >
                    <input 
                        type="number" 
                        name="page" 
                        min="1" 
                        max="{{ $paginator->lastPage() }}" 
                        class="w-16 px-2 py-1 border rounded-l text-sm dark:bg-gray-800 dark:border-gray-600"
                        placeholder="{{ $paginator->currentPage() }}"
                        aria-label="Page number"
                    >
                    <button 
                        type="submit"
                        class="bg-primary-500 text-white px-3 py-1 rounded-r text-sm hover:bg-primary-600"
                    >
                        Go
                    </button>
                </form>
            </div>
        </div>
    </div>
@endif
```

## Key Features Added

1. **Result Count Display**:
   ```blade
   Showing {{ $paginator->firstItem() }} to {{ $paginator->lastItem() }} of {{ $paginator->total() }} results
   ```

2. **Navigation Controls**:
   - First Page (`&laquo;`)
   - Previous Page (`&lsaquo;`)
   - Next Page (`&rsaquo;`)
   - Last Page (`&raquo;`)

3. **Page Number Buttons**:
   - Current page highlighted
   - Clickable page numbers
   - Ellipsis for large page ranges

4. **Go to Page**:
   ```blade
   <form wire:submit.prevent="gotoPage($event.target.page.value)">
     <input type="number" min="1" max="{{ $paginator->lastPage() }}">
   </form>
   ```

## Customization Options

### Change Colors
Modify these classes in the code:
```blade
'bg-primary-500 text-white'  // Current page
'bg-gray-100 dark:bg-gray-700'  // Hover state
```

### Adjust Spacing
Modify these values:
```blade
class="px-3 py-1"  // Button padding
class="ml-4"       // Input margin
class="w-16"       // Input width
```

### Add Icons
Replace text arrows with Heroicons:
```blade
First Page: <x-heroicon-o-arrow-left-circle class="w-5 h-5" />
Last Page: <x-heroicon-o-arrow-right-circle class="w-5 h-5" />
```

## Common Issues & Solutions

### 1. Pagination File Not Created
**Solution**: Manually create the file path:
```bash
mkdir -p resources/views/vendor/filament/components/table
touch resources/views/vendor/filament/components/table/pagination.blade.php
```

### 2. Styles Not Applying
**Fix**: Add dark mode support with `dark:` variants:
```blade
class="text-gray-700 dark:text-gray-300"
```

### 3. Go to Page Not Working
**Ensure**:
- Input has `name="page"`
- Form uses `wire:submit.prevent`
- Min/max attributes are set:
  ```blade
  min="1" max="{{ $paginator->lastPage() }}"
  ```

### 4. Mobile Responsiveness
The included code uses:
```blade
<div class="flex flex-col md:flex-row">
```
This stacks elements vertically on mobile and horizontally on desktop.

## Alternative Components

For more advanced needs, consider these Filament plugins:
1. [Filament Pagination](https://filamentphp.com/plugins/riadh-pagination)
2. [Custom Table Components](https://filamentphp.com/plugins/filament-custom-table-component)

## Livewire Pagination Methods Used

| Method | Description |
|--------|-------------|
| `gotoPage($page)` | Jump to specific page |
| `previousPage()` | Go to previous page |
| `nextPage()` | Go to next page |

The paginator object provides:
- `currentPage()`
- `lastPage()`
- `firstItem()`
- `lastItem()`
- `total()`
- `hasPages()`
- `onFirstPage()`
- `hasMorePages()`

```

This README includes:
1. Step-by-step implementation guide
2. Complete code solution with comments
3. Customization options
4. Troubleshooting section for common issues
5. Responsive design considerations
6. Alternative plugin recommendations
7. Livewire method reference

Key additions beyond basic pagination:
- Dark mode support
- Mobile responsiveness
- ARIA labels for accessibility
- Visual feedback for disabled states
- Hover effects
- Current page highlighting
- Input validation (min/max page numbers)
- Error prevention for manual file creation
