# Large Employee Form with Sections


<img src="https://filamentexamples.com/storage/170/01J81YJEQD1WGSNDDP5TZN1RKY.png" />
 

source : https://filamentexamples.com/project/large-employee-form-with-tabs

code : https://github.com/LaravelDaily/FilamentExamples-Projects/tree/main/forms/large-employee-form-with-tabs



This project demonstrates these Filament form features:

1. Setting settings globally
2. Usage of grid and group to show inputs in different positions

The repository contains the complete Laravel + Filament project to demonstrate the functionality, including migrations/seeds for the demo data.

The Filament project is in the `app/Filament` folder.

Feel free to pick the parts that you actually need in your projects.

---

## How to install

- Clone the repository with `git clone`
- Copy the `.env.example` file to `.env` and edit database credentials there
- Run `composer install`
- Run `php artisan key:generate`
- Run `php artisan storage:link`
- Run `php artisan migrate --seed` (it has some seeded data for your testing)
- That's it: launch the URL `/admin` and log in with credentials `admin@admin.com` and `password`. Then go to `Employees` and click `New Post`.

---

## Screenshot

![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-1.png)
![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-2.png)
![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-3.png)
![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-4.png)
![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-5.png)
![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-6.png)
![](https://laraveldaily.com/uploads/2024/09/big-emplotyee-form-with-tabs-7.png)

---

## How It Works

First, we have set global settings for some inputs and sections. This means we do not need to set an option for every input.

**app/Providers/AppServiceProvider.php**:
```php
use Filament\Forms\Components\Radio;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\Section;
use Illuminate\Support\ServiceProvider;
use Filament\Forms\Components\Tabs\Tab;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\DatePicker;

class AppServiceProvider extends ServiceProvider
{
    // ...
    
    public function boot(): void
    {
        TextInput::configureUsing(function (TextInput $textInput) {
            $textInput->inlineLabel();
        });

        Radio::configureUsing(function (Radio $radio) {
            $radio->inlineLabel();
        });

        Select::configureUsing(function (Select $select) {
            $select->inlineLabel();
        });

        DatePicker::configureUsing(function (DatePicker $datePicker) {
            $datePicker->inlineLabel();
        });

        Tab::configureUsing(function (Tab $section) {
            $section->columns();
        });
    }
}
```

Next, we will go through every tab.

The first tab, `Personal Information,` has basic text inputs, a date picker, and a selector. The social security number is a mask that only allows numbers and is divided by a dash. The payroll number is shown as a placeholder and is set to be visible only on the edit page.

```php
Tabs\Tab::make('Personal Information')
    ->schema([
        Forms\Components\TextInput::make('first_name')
            ->required(),
        Forms\Components\TextInput::make('middle_name'),
        Forms\Components\TextInput::make('last_name')
            ->required(),
        Forms\Components\TextInput::make('social_security_number')
            ->mask('999-99-9999')
            ->placeholder('000-000-0000')
            ->required(),
        Forms\Components\DatePicker::make('birth_date')
            ->label('Date Of Birth')
            ->required(),
        Forms\Components\Select::make('gender')
            ->options([
                'Male' => 'Male',
                'Female' => 'Female',
                'Will Not Answer' => 'Will Not Answer',
            ])
            ->default('Male'),
        Forms\Components\Placeholder::make('id')
            ->label('Payroll Number')
            ->inlineLabel()
            ->visibleOn('edit'),
    ]),
```

The `Contact Information` has inputs without any specific changes. The state is tied to a relationship.

```php
Tabs\Tab::make('Contact Information')
    ->schema([
        Forms\Components\TextInput::make('address'),
        Forms\Components\TextInput::make('city'),
        Forms\Components\Select::make('state_id')
            ->relationship('state', 'name'),
        Forms\Components\TextInput::make('zip_code'),
        Forms\Components\TextInput::make('phone'),
        Forms\Components\TextInput::make('cell_phone'),
        Forms\Components\TextInput::make('emergency_contact_phone'),
        Forms\Components\TextInput::make('email')
            ->email('email'),
        Forms\Components\TextInput::make('emergency_contact_name'),
        Forms\Components\TextInput::make('emergency_contact_relationship'),
    ]),
```

The `Employment Information` has inputs without any specific changes. Store and position are tied to a relationship.

```php
Tabs\Tab::make('Employment Information')
    ->schema([
        Forms\Components\Select::make('store_id')
            ->required()
            ->label('Store')
            ->relationship('store', 'name'),
        Forms\Components\DatePicker::make('date_hired')
            ->required(),
        Forms\Components\Select::make('position_id')
            ->required()
            ->label('Position')
            ->relationship('position', 'name'),
        Forms\Components\Radio::make('status')
            ->required()
            ->default('active')
            ->inline()
            ->options([
                'active' => 'Active',
                'inactive' => 'Inactive',
                'new' => 'New',
            ]),
        Forms\Components\TextInput::make('pay_rate'),
        Forms\Components\TextInput::make('payroll_group'),
        Forms\Components\TextInput::make('work_classification')
            ->required(),
    ]),
```

In the `EFT Information`, we have a `Bank` input that is added to a grid. The grid is set to two columns by default. Because we have two columns and one input, it fills the first grid column.

```php
Tabs\Tab::make('EFT Information')
    ->schema([
        Forms\Components\TextInput::make('pay_method')
            ->required(),
        Forms\Components\TextInput::make('account_type')
            ->required(),
        Forms\Components\Grid::make()
            ->schema([
                Forms\Components\TextInput::make('bank')
                    ->required(),
            ]),
        Forms\Components\TextInput::make('routing_number')
            ->required(),
        Forms\Components\TextInput::make('account_number')
            ->required(),
    ]),
```

In the `Deduction` section, the last input for the `Vision` is added to a grid. This grid item is set to start at the second grid column.

```php
Tabs\Tab::make('Deduction')
    ->schema([
        Forms\Components\TextInput::make('deduction_garnishment')
            ->label('Garnishment'),
        Forms\Components\TextInput::make('deduction_health_insurance')
            ->label('Health Insurance'),
        Forms\Components\TextInput::make('deduction_child_support')
            ->label('Child Support'),
        Forms\Components\TextInput::make('deduction_dental')
            ->label('Dental'),
        Forms\Components\Grid::make()
            ->schema([
                Forms\Components\TextInput::make('deduction_vision')
                    ->label('Vision')
                    ->columnStart(2),
            ]),
    ]),
```

The `I9 / W4 Information` section has the most logic in this form.

The `Citizenship` is added to a group with columns set to ten. Inside this group, we have two additional groups: one for radio select and the other for two date pickers, which are aligned to the bottom of the group.

These two date pickers have a validation rule that is required if citizenship has a value of `permanent_resident` or `authorized_to_work`.

Next, we have the `Other Adjustments` field, which has three inputs attached to it. To achieve this result, everything is added to a group, and inside this group, we have two other groups, one for the `Other Adjustments` label and the other for three text inputs.

The label is added as a placeholder and centered within its group. The other three text inputs have the validation rule `requiredWithoutAll`, which allows one of the inputs to be required.

The `Tax credit survey` text input is added to a grid so that it would be shown in the first half of the line.

```php
Tabs\Tab::make('I9 / W4 Information')
    ->schema([
        Forms\Components\Group::make([
            Forms\Components\Group::make([
                Forms\Components\Radio::make('us_citizenship')
                    ->label('Citizenship')
                    ->options([
                        'citizen' => 'Citizen',
                        'non_citizen' => 'Non Citizen',
                        'permanent_resident' => 'Permanent Citizen',
                        'authorized_to_work' => 'Authorized to Work',
                    ])
                    ->default('citizen')
                    ->required(),
            ])->columnSpan(3),
            Forms\Components\Group::make([
                Forms\Components\DatePicker::make('resident_until')
                    ->requiredIf('us_citizenship', 'permanent_citizen')
                    ->hiddenLabel(),
                Forms\Components\DatePicker::make('authorized_to_work_until')
                    ->requiredIf('us_citizenship', 'authorized_to_work')
                    ->hiddenLabel(),
            ])->extraAttributes(['class' => 'h-full flex flex-col justify-end'])
                ->columnSpan(7),
        ])
            ->columns(10)
            ->columnSpanFull(),

        Forms\Components\Select::make('tax_filing_status')
            ->required()
            ->options([
                'single' => 'Single',
                'married' => 'Married',
                'filing_separately' => 'Filing Separately',
                'filing_jointly' => 'Filing Jointly',
                'head_of_household' => 'Head of Household',
            ])
            ->default('single'),
        Forms\Components\Checkbox::make('has_multiple_jobs')
            ->label('Multiple Jobs or Spouse Works'),
        Forms\Components\TextInput::make('claim_dependents')
            ->prefix('$')
            ->numeric()
            ->default(0),

        Forms\Components\Group::make([
            Forms\Components\Group::make([
                Forms\Components\Placeholder::make('other_adjustments')
                    ->hiddenLabel()
                    ->content(new HtmlString('<span class="text-sm font-medium leading-6 text-gray-950 dark:text-white">Other Adjustments</span> <sup class="text-danger-600 dark:text-danger-400 font-medium">*</sup>')),
            ])->extraAttributes(['class' => 'h-full flex items-center']),
            Forms\Components\Group::make([
                Forms\Components\TextInput::make('tax_other_income')
                    ->label('Other Income')
                    ->requiredWithoutAll('tax_deductions,tax_extra_withholding'),
                Forms\Components\TextInput::make('tax_deductions')
                    ->label('Deductions')
                    ->requiredWithoutAll('tax_other_income,tax_extra_withholding'),
                Forms\Components\TextInput::make('tax_extra_withholding')
                    ->label('Extra Withholding')
                    ->requiredWithoutAll('tax_deductions,tax_other_income'),
            ])->columnSpan(2)
        ])->columns(3),

        Forms\Components\Grid::make()
            ->schema([
                Forms\Components\TextInput::make('tax_credit_survey'),
            ]),
        Forms\Components\Radio::make('is_tax_exempted')
            ->label('Tax Exempted')
            ->inline()
            ->options([
                'yes' => 'Yes',
                'no' => 'No',
            ])
            ->default('no'),
    ]),
```

The `General Information` section has one group, inside which are two separate grid sections, each with two columns by default. Each grid has a radio input, which is shown only in the first grid column.

```php
Tabs\Tab::make('General Information')
    ->schema([
        Forms\Components\TextInput::make('auto_allowance')
            ->numeric()
            ->default(0),
        Forms\Components\TextInput::make('phone_allowance')
            ->numeric()
            ->default(0),
        Forms\Components\Radio::make('is_serv_safe')
            ->label('Serv Safe')
            ->inline()
            ->options([
                'yes' => 'Yes',
                'no' => 'No',
            ])
            ->default('no'),
        Forms\Components\Radio::make('is_food_handler')
            ->label('Food Handler')
            ->inline()
            ->options([
                'yes' => 'Yes',
                'no' => 'No',
            ])
            ->default('no'),
        Forms\Components\Group::make([
            Forms\Components\Grid::make()
                ->schema([
                    Forms\Components\Radio::make('has_background_check')
                        ->label('Background Check Completed')
                        ->inline()
                        ->options([
                            'yes' => 'Yes',
                            'no' => 'No',
                        ])
                        ->default('yes'),
                ]),
            Forms\Components\Grid::make()
                ->schema([
                    Forms\Components\Radio::make('has_paper_work')
                        ->required()
                        ->label('Submitted All Paper Work')
                        ->inline()
                        ->options([
                            'yes' => 'Yes',
                            'no' => 'No',
                        ])
                        ->default('yes'),
                ])
        ])->columnSpanFull(),
        Forms\Components\Radio::make('is_eligible_for_rehire')
            ->label('Eligible For Rehire')
            ->inline()
            ->options([
                'yes' => 'Yes',
                'no' => 'No',
            ])
            ->default('yes'),
        Forms\Components\TextInput::make('eligible_for_rehire_reason'),
        Forms\Components\Textarea::make('comments')
            ->columnSpanFull(),
        Forms\Components\DatePicker::make('last_worked'),
    ]),
]),
```

This is how the complete form looks like:

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Forms\Components\Tabs::make()
                ->columnSpanFull()
                ->tabs([
                    Tabs\Tab::make('Personal Information')
                        ->schema([
                            Forms\Components\TextInput::make('first_name')
                                ->required(),
                            Forms\Components\TextInput::make('middle_name'),
                            Forms\Components\TextInput::make('last_name')
                                ->required(),
                            Forms\Components\TextInput::make('social_security_number')
                                ->mask('999-99-9999')
                                ->placeholder('000-000-0000')
                                ->required(),
                            Forms\Components\DatePicker::make('birth_date')
                                ->label('Date Of Birth')
                                ->required(),
                            Forms\Components\Select::make('gender')
                                ->options([
                                    'Male' => 'Male',
                                    'Female' => 'Female',
                                    'Will Not Answer' => 'Will Not Answer',
                                ])
                                ->default('Male'),
                            Forms\Components\Placeholder::make('id')
                                ->label('Payroll Number')
                                ->inlineLabel()
                                ->visibleOn('edit'),
                        ]),

                    Tabs\Tab::make('Contact Information')
                        ->schema([
                            Forms\Components\TextInput::make('address'),
                            Forms\Components\TextInput::make('city'),
                            Forms\Components\Select::make('state_id')
                                ->relationship('state', 'name'),
                            Forms\Components\TextInput::make('zip_code'),
                            Forms\Components\TextInput::make('phone'),
                            Forms\Components\TextInput::make('cell_phone'),
                            Forms\Components\TextInput::make('emergency_contact_phone'),
                            Forms\Components\TextInput::make('email')
                                ->email('email'),
                            Forms\Components\TextInput::make('emergency_contact_name'),
                            Forms\Components\TextInput::make('emergency_contact_relationship'),
                        ]),

                    Tabs\Tab::make('Employment Information')
                        ->schema([
                            Forms\Components\Select::make('store_id')
                                ->required()
                                ->label('Store')
                                ->relationship('store', 'name'),
                            Forms\Components\DatePicker::make('date_hired')
                                ->required(),
                            Forms\Components\Select::make('position_id')
                                ->required()
                                ->label('Position')
                                ->relationship('position', 'name'),
                            Forms\Components\Radio::make('status')
                                ->required()
                                ->default('active')
                                ->inline()
                                ->options([
                                    'active' => 'Active',
                                    'inactive' => 'Inactive',
                                    'new' => 'New',
                                ]),
                            Forms\Components\TextInput::make('pay_rate'),
                            Forms\Components\TextInput::make('payroll_group'),
                            Forms\Components\TextInput::make('work_classification')
                                ->required(),
                        ]),

                    Tabs\Tab::make('EFT Information')
                        ->schema([
                            Forms\Components\TextInput::make('pay_method')
                                ->required(),
                            Forms\Components\TextInput::make('account_type')
                                ->required(),
                            // Maybe could be done better? Idk lol. This was faster.
                            Forms\Components\Grid::make()
                                ->schema([
                                    Forms\Components\TextInput::make('bank')
                                        ->required(),
                                ]),
                            Forms\Components\TextInput::make('routing_number')
                                ->required(),
                            Forms\Components\TextInput::make('account_number')
                                ->required(),
                        ]),

                    Tabs\Tab::make('Deduction')
                        ->schema([
                            Forms\Components\TextInput::make('deduction_garnishment')
                                ->label('Garnishment'),
                            Forms\Components\TextInput::make('deduction_health_insurance')
                                ->label('Health Insurance'),
                            Forms\Components\TextInput::make('deduction_child_support')
                                ->label('Child Support'),
                            Forms\Components\TextInput::make('deduction_dental')
                                ->label('Dental'),
                            Forms\Components\Grid::make()
                                ->schema([
                                    Forms\Components\TextInput::make('deduction_vision')
                                        ->label('Vision')
                                        ->columnStart(2),
                                ]),
                        ]),

                    Tabs\Tab::make('I9 / W4 Information')
                        ->schema([
                            Forms\Components\Group::make([
                                Forms\Components\Group::make([
                                    Forms\Components\Radio::make('us_citizenship')
                                        ->label('Citizenship')
                                        ->options([
                                            'citizen' => 'Citizen',
                                            'non_citizen' => 'Non Citizen',
                                            'permanent_resident' => 'Permanent Citizen',
                                            'authorized_to_work' => 'Authorized to Work',
                                        ])
                                        ->default('citizen')
                                        ->required(),
                                ])->columnSpan(3),
                                Forms\Components\Group::make([
                                    Forms\Components\DatePicker::make('resident_until')
                                        ->requiredIf('us_citizenship', 'permanent_citizen')
                                        ->hiddenLabel(),
                                    Forms\Components\DatePicker::make('authorized_to_work_until')
                                        ->requiredIf('us_citizenship', 'authorized_to_work')
                                        ->hiddenLabel(),
                                ])->extraAttributes(['class' => 'h-full flex flex-col justify-end'])
                                    ->columnSpan(7),
                            ])
                                ->columns(10)
                                ->columnSpanFull(),

                            Forms\Components\Select::make('tax_filing_status')
                                ->required()
                                ->options([
                                    'single' => 'Single',
                                    'married' => 'Married',
                                    'filing_separately' => 'Filing Separately',
                                    'filing_jointly' => 'Filing Jointly',
                                    'head_of_household' => 'Head of Household',
                                ])
                                ->default('single'),
                            Forms\Components\Checkbox::make('has_multiple_jobs')
                                ->label('Multiple Jobs or Spouse Works'),
                            Forms\Components\TextInput::make('claim_dependents')
                                ->prefix('$')
                                ->numeric()
                                ->default(0),

                            Forms\Components\Group::make([
                                Forms\Components\Group::make([
                                    Forms\Components\Placeholder::make('other_adjustments')
                                        ->hiddenLabel()
                                        ->content(new HtmlString('<span class="text-sm font-medium leading-6 text-gray-950 dark:text-white">Other Adjustments</span> <sup class="text-danger-600 dark:text-danger-400 font-medium">*</sup>')),
                                ])->extraAttributes(['class' => 'h-full flex items-center']),
                                Forms\Components\Group::make([
                                    Forms\Components\TextInput::make('tax_other_income')
                                        ->label('Other Income')
                                        ->requiredWithoutAll('tax_deductions,tax_extra_withholding'),
                                    Forms\Components\TextInput::make('tax_deductions')
                                        ->label('Deductions')
                                        ->requiredWithoutAll('tax_other_income,tax_extra_withholding'),
                                    Forms\Components\TextInput::make('tax_extra_withholding')
                                        ->label('Extra Withholding')
                                        ->requiredWithoutAll('tax_deductions,tax_other_income'),
                                ])->columnSpan(2)
                            ])->columns(3),

                            Forms\Components\Grid::make()
                                ->schema([
                                    Forms\Components\TextInput::make('tax_credit_survey'),
                                ]),
                            Forms\Components\Radio::make('is_tax_exempted')
                                ->label('Tax Exempted')
                                ->inline()
                                ->options([
                                    'yes' => 'Yes',
                                    'no' => 'No',
                                ])
                                ->default('no'),
                        ]),

                    Tabs\Tab::make('General Information')
                        ->schema([
                            Forms\Components\TextInput::make('auto_allowance')
                                ->numeric()
                                ->default(0),
                            Forms\Components\TextInput::make('phone_allowance')
                                ->numeric()
                                ->default(0),
                            Forms\Components\Radio::make('is_serv_safe')
                                ->label('Serv Safe')
                                ->inline()
                                ->options([
                                    'yes' => 'Yes',
                                    'no' => 'No',
                                ])
                                ->default('no'),
                            Forms\Components\Radio::make('is_food_handler')
                                ->label('Food Handler')
                                ->inline()
                                ->options([
                                    'yes' => 'Yes',
                                    'no' => 'No',
                                ])
                                ->default('no'),
                            Forms\Components\Group::make([
                                Forms\Components\Grid::make()
                                    ->schema([
                                        Forms\Components\Radio::make('has_background_check')
                                            ->label('Background Check Completed')
                                            ->inline()
                                            ->options([
                                                'yes' => 'Yes',
                                                'no' => 'No',
                                            ])
                                            ->default('yes'),
                                    ]),
                                Forms\Components\Grid::make()
                                    ->schema([
                                        Forms\Components\Radio::make('has_paper_work')
                                            ->required()
                                            ->label('Submitted All Paper Work')
                                            ->inline()
                                            ->options([
                                                'yes' => 'Yes',
                                                'no' => 'No',
                                            ])
                                            ->default('yes'),
                                    ])
                            ])->columnSpanFull(),
                            Forms\Components\Radio::make('is_eligible_for_rehire')
                                ->label('Eligible For Rehire')
                                ->inline()
                                ->options([
                                    'yes' => 'Yes',
                                    'no' => 'No',
                                ])
                                ->default('yes'),
                            Forms\Components\TextInput::make('eligible_for_rehire_reason'),
                            Forms\Components\Textarea::make('comments')
                                ->columnSpanFull(),
                            Forms\Components\DatePicker::make('last_worked'),
                        ]),
                ]),
        ]);
}
```

