# general-guidance v1.0

=================================================================

- [Code Writing Guidelines](#code-writing-guidelines)
- [Table Naming Guidelines](#table-naming-guidelines)
- [UI/UX](#ui-ux)
- [Version Control](#version-control)
- [API/Stored Procedure Documentation](#api-docs)
- [Scheduler Documentation](#sch-docs)
- [Penetration Test Guidance](#pentest-docs)

=================================================================

## Code Writing Guidelines
### Content1

1. [Prinsip *single responsibility*](#prinsip-single-responsibility)
2. [Model tebal, *controller* tipis](#model-tebal-controller-tipis)
3. [Validasi](#validasi)
4. [*Business logic* harus di dalam kelas *services*](#business-logic-harus-di-dalam-kelas-services)
5. [*Don't repeat yourself* (DRY)](#dont-repeat-yourself-dry)
6. [Lebih memilih menggunakan *Eloquent* atau *Query Builder*  daripada menggunakan query SQL mentah. Lebih memilih *collections* daripada *array*](#lebih-memilih-menggunakan-eloquent-daripada-menggunakan-query-builder-dan-query-sql-mentah-lebih-memilih-collections-daripada-array)
7. [*Mass assignment*](#mass-assignment)
8. [Jangan mengeksekusi kueri dalam *template blade* dan gunakan *eager loading* (masalah N + 1)](#jangan-mengeksekusi-kueri-dalam-template-blade-dan-gunakan-eager-loading-masalah-n-1)
9. [Komentari kode anda, tetapi lebih baik *method* dan nama variabel yang deskriptif daripada komentar](#komentari-kode-anda-tetapi-lebih-baik-method-dan-nama-variabel-yang-deskriptif-daripada-komentar)
10. [Jangan letakkan JS dan CSS di *template blade* dan jangan letakkan HTML apa pun di kelas PHP](#jangan-letakkan-js-dan-css-di-template-blade-dan-jangan-letakkan-html-apa-pun-di-kelas-php)
11. [Gunakan file *config*, *language*, dan konstanta daripada teks dalam kode](#gunakan-file-config-language-dan-konstanta-daripada-teks-dalam-kode)
12. [Gunakan *tools* standar Laravel yang diterima oleh komunitas](#gunakan-tools-standar-laravel-yang-diterima-oleh-komunitas)
13. [Ikuti konvensi penamaan Laravel](#ikuti-konvensi-penamaan-laravel)
14. [Gunakan sintaks yang lebih pendek dan lebih mudah dibaca jika memungkinkan](#gunakan-sintaks-yang-lebih-pendek-dan-lebih-mudah-dibaca-jika-memungkinkan)
15. [Gunakan *IoC Container* atau *facades* daripada kelas baru](#gunakan-ioc-container-atau-facades-daripada-kelas-baru)
16. [Jangan mendapatkan data dari file `.env` secara langsung](#jangan-mendapatkan-data-dari-file-env-secara-langsung)
17. [Simpan tanggal dalam format standar. Gunakan *accessors* dan *mutators* untuk mengubah format tanggal](#simpan-tanggal-dalam-format-standar-gunakan-accessors-dan-mutators-untuk-mengubah-format-tanggal)
18. [Praktik bagus lainnya](#praktik-bagus-lainnya)

### **Prinsip *single responsibility***

Kelas dan metode seharusnya hanya memiliki satu tanggung jawab.

Contoh buruk:

```php
public function getFullNameAttribute(): string
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

Contoh terbaik:

```php
public function getFullNameAttribute(): string
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient(): bool
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong(): string
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort(): string
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

[back to top](#content1)

### **Model tebal, *controller* tipis**

Masukkan semua logika terkait DB ke model *eloquent* atau ke dalam kelas repositori jika anda menggunakan *Query Builder* atau kueri SQL mentah.

Contoh buruk:

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

Contoh terbaik:

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

[back to top](#content1)

### **Validasi**

Pindahkan validasi dari *controller* ke kelas *request*.

Contoh buruk:

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ...
}
```

Contoh terbaik:

```php
public function store(PostRequest $request)
{
    ...
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```

[back to top](#content1)

### ***Business logic* harus di dalam kelas *services***

*Controller* harus hanya memiliki satu tanggung jawab, jadi pindahkan *business logic* dari *controller* ke kelas *service*.

Contoh buruk:

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }
    
    ...
}
```

Contoh terbaik:

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ...
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```

[back to top](#content1)

### ***Don't repeat yourself* (DRY)**

Gunakan kembali kode ketika anda bisa. [PSR](#prinsip-single-responsibility) membantu anda menghindari duplikasi. Juga, gunakan kembali *template blade*, *scope eloquent*, dll.

Contoh buruk:

```php
public function getActive()
{
    return $this->where('verified', 1)->whereNotNull('deleted_at')->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->where('verified', 1)->whereNotNull('deleted_at');
        })->get();
}
```

Contoh terbaik:

```php
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```

[back to top](#content1)

### **Lebih memilih menggunakan *Eloquent* daripada menggunakan *Query Builder* dan query SQL mentah. Lebih memilih *collections* daripada *array***

*Eloquent* memungkinkan anda menulis kode yang dapat dibaca dan *maintainable*. Dan, *Eloquent* memiliki *built-in tools* yang bagus seperti *soft deletes*, *events*, *scopes*, dll.

Contoh buruk:

```sql
SELECT *
FROM `articles`
WHERE EXISTS (SELECT *
              FROM `users`
              WHERE `articles`.`user_id` = `users`.`id`
              AND EXISTS (SELECT *
                          FROM `profiles`
                          WHERE `profiles`.`user_id` = `users`.`id`) 
              AND `users`.`deleted_at` IS NULL)
AND `verified` = '1'
AND `active` = '1'
ORDER BY `created_at` DESC
```

Contoh terbarik 1:
```php
$products = DB::table('products')
    ->select('id', 'name', 'price')
    ->orderBy('price', 'asc')
    ->get();
```

Contoh terbaik 2:

```php
Article::has('user.profile')->verified()->latest()->get();
```



[back to top](#content1)

### ***Mass assignment***

Contoh buruk:

```php
$article = new Article;
$article->title = $request->title;
$article->content = $request->content;
$article->verified = $request->verified;

// Add category to article
$article->category_id = $category->id;
$article->save();
```

Contoh terbaik:

```php
$category->article()->create($request->validated());
```

[back to top](#content1)

### **Jangan mengeksekusi kueri dalam *template blade* dan gunakan *eager loading* (masalah N + 1)**

Contoh buruk (untuk 100 *user*, 101 kueri DB akan dieksekusi):

```blade
@foreach (User::all() as $user)
    {{ $user->profile->name }}
@endforeach
```

Contoh terbaik (untuk 100 *user*, 2 kueri DB akan dieksekusi):

```php
$users = User::with('profile')->get();

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

[back to top](#content1)

### **Komentari kode anda, tetapi lebih baik *method* dan nama variabel yang deskriptif daripada komentar**

Contoh buruk:

```php
if (count((array) $builder->getQuery()->joins) > 0)
```

Contoh lebih baik:

```php
// Determine if there are any joins.
if (count((array) $builder->getQuery()->joins) > 0)
```

Contoh terbaik:

```php
if ($this->hasJoins())
```

[back to top](#content1)

### **Jangan letakkan JS dan CSS di *template blade* dan jangan letakkan HTML apa pun di kelas PHP**

Contoh buruk:

```javascript
let article = `{{ json_encode($article) }}`;
```

Contoh lebih baik:

```php
<input id="article" type="hidden" value='@json($article)'>

Atau

<button class="js-fav-article" data-article='@json($article)'>{{ $article->name }}<button>
```

Dalam file javascript:

```javascript
let article = $('#article').val();
```

Cara terbaik adalah dengan menggunakan *package* `PHP to JS` khusus untuk mentransfer data.

[back to top](#content1)

### **Gunakan file *config*, *language*, dan konstanta daripada teks dalam kode**

Contoh buruk:

```php
public function isNormal()
{
    return $article->type === 'normal';
}

return back()->with('message', 'Your article has been added!');
```

Contoh terbaik:

```php
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```

[back to top](#content1)

### **Gunakan *tools* standar Laravel yang diterima oleh komunitas**

Selalu gunakan fungsi *built-in* bawaan laravel dan *packages* komunitas daripada menggunakan *packages* dan *tools* pihak ke-3. *Developer* manapun yang akan bekerja dengan aplikasi anda di masa mendatang perlu mempelajari *tools* baru. Dan juga, peluang untuk mendapatkan bantuan dari komunitas Laravel jauh lebih rendah saat anda menggunakan *packages* atau *tools* pihak ke-3. Jangan membuat klien Anda membayar untuk itu.

Task | Tools *standar* | *Tools* pihak ke-3
------------ | ------------- | -------------
Authorization | Policies | Entrust, Sentinel and other packages
Compiling assets | Laravel Mix, Vite | Grunt, Gulp, 3rd party packages
Development Environment | Laravel Sail, Homestead | Docker
Deployment | Laravel Forge | Deployer and other solutions
Unit testing | PHPUnit, Mockery | Phpspec, Pest
Browser testing | Laravel Dusk | Codeception
DB | Eloquent | SQL, Doctrine
Templates | Blade | Twig
Working with data | Laravel collections | Arrays
Form validation | Request classes | 3rd party packages, validation in controller
Authentication | Built-in | 3rd party packages, your own solution
API authentication | Laravel Passport, Laravel Sanctum | 3rd party JWT and OAuth packages
Creating API | Built-in | Dingo API and similar packages
Working with DB structure | Migrations | Working with DB structure directly
Localization | Built-in | 3rd party packages
Realtime user interfaces | Laravel Echo, Pusher | 3rd party packages and working with WebSockets directly
Generating testing data | Seeder classes, Model Factories, Faker | Creating testing data manually
Task scheduling | Laravel Task Scheduler | Scripts and 3rd party packages
DB | MySQL, PostgreSQL, SQLite, SQL Server | MongoDB

[back to top](#content1)

### **Ikuti konvensi penamaan Laravel**

Ikuti [PSR standards](https://www.php-fig.org/psr/psr-12/).

Dan juga, ikuti konvensi penamaan yang diterima oleh komunitas Laravel:

What | How | Good | Bad
------------ | ------------- | ------------- | -------------
Controller | singular | ArticleController | ~~ArticlesController~~
Route | plural | articles/1 | ~~article/1~~
Route name | snake_case with dot notation | users.show_active | ~~users.show-active, show-active-users~~
Model | singular | User | ~~Users~~
hasOne or belongsTo relationship | singular | articleComment | ~~articleComments, article_comment~~
All other relationships | plural | articleComments | ~~articleComment, article_comments~~
Table | plural | article_comments | ~~article_comment, articleComments~~
Pivot table | singular model names in alphabetical order | article_user | ~~user_article, articles_users~~
Table column | snake_case without model name | meta_title | ~~MetaTitle; article_meta_title~~
Model property | snake_case | $model->created_at | ~~$model->createdAt~~
Foreign key | singular model name with _id suffix | article_id | ~~ArticleId, id_article, articles_id~~
Primary key | - | id | ~~custom_id~~
Migration | - | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~
Method | camelCase | getAll | ~~get_all~~
Method in resource controller | [table](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~
Method in test class | camelCase | testGuestCannotSeeArticle | ~~test_guest_cannot_see_article~~
Variable | camelCase | $articlesWithAuthor | ~~$articles_with_author~~
Collection | descriptive, plural | $activeUsers = User::active()->get() | ~~$active, $data~~
Object | descriptive, singular | $activeUser = User::active()->first() | ~~$users, $obj~~
Config and language files index | snake_case | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
View | kebab-case | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
Config | snake_case | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~
Contract (interface) | adjective or noun | AuthenticationInterface | ~~Authenticatable, IAuthentication~~
Trait | adjective | Notifiable | ~~NotificationTrait~~
Trait [(PSR)](https://www.php-fig.org/bylaws/psr-naming-conventions/) | adjective | NotifiableTrait | ~~Notification~~
Enum | singular | UserType | ~~UserTypes~~, ~~UserTypeEnum~~
FormRequest | singular | UpdateUserRequest | ~~UpdateUserFormRequest~~, ~~UserFormRequest~~, ~~UserRequest~~
Seeder | singular | UserSeeder | ~~UsersSeeder~~

[back to top](#content1)

### **Gunakan sintaks yang lebih pendek dan lebih mudah dibaca jika memungkinkan**

Contoh buruk:

```php
$request->session()->get('cart');
$request->input('name');
```

Contoh terbaik:

```php
session('cart');
$request->name;
```

Contoh:

Sintaks umum | Sintaks pendek dan mudah dibaca
------------ | -------------
`Session::get('cart')` | `session('cart')`
`$request->session()->get('cart')` | `session('cart')`
`Session::put('cart', $data)` | `session(['cart' => $data])`
`$request->input('name'), Request::get('name')` | `$request->name, request('name')`
`return Redirect::back()` | `return back()`
`is_null($object->relation) ? null : $object->relation->id` | `optional($object->relation)->id`
`return view('index')->with('title', $title)->with('client', $client)` | `return view('index', compact('title', 'client'))`
`$request->has('value') ? $request->value : 'default';` | `$request->get('value', 'default')`
`Carbon::now(), Carbon::today()` | `now(), today()`
`App::make('Class')` | `app('Class')`
`->where('column', '=', 1)` | `->where('column', 1)`
`->orderBy('created_at', 'desc')` | `->latest()`
`->orderBy('age', 'desc')` | `->latest('age')`
`->orderBy('created_at', 'asc')` | `->oldest()`
`->select('id', 'name')->get()` | `->get(['id', 'name'])`
`->first()->name` | `->value('name')`

[back to top](#content1)

### **Gunakan *IoC Container* atau *facades* daripada kelas baru**

Sintaks Kelas baru membuat penggabungan yang sempit antar kelas dan memperumit proses pengujian. Gunakan *facades* atau *IoC container* sebagai gantinya.

Contoh buruk:

```php
$user = new User;
$user->create($request->validated());
```

Contoh terbaik:

```php
public function __construct(User $user)
{
    $this->user = $user;
}

...

$this->user->create($request->validated());
```

[back to top](#content1)

### **Jangan mendapatkan data dari file `.env` secara langsung**

Alihkan data ke file konfigurasi sebagai gantinya dan kemudian gunakan fungsi `config ()` untuk menggunakan data dalam aplikasi.

Contoh buruk:

```php
$apiKey = env('API_KEY');
```

Contoh terbaik:

```php
// config/api.php
'key' => env('API_KEY'),

// Use the data
$apiKey = config('api.key');
```

[back to top](#content1)

### **Simpan tanggal dalam format standar. Gunakan *accessors* dan *mutators* untuk mengubah format tanggal**

Contoh buruk:

```php
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->toDateString() }}
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->format('m-d') }}
```

Contoh terbaik:

```php
// Model
protected $casts = [
    'ordered_at' => 'datetime',
];

public function getSomeDateAttribute($date)
{
    return $date->format('m-d');
}

// View
{{ $object->ordered_at->toDateString() }}
{{ $object->ordered_at->some_date }}
```

[back to top](#content1)

### **Praktik bagus lainnya**

Jangan pernah menaruh logika apa pun di file `route`.

Minimalkan penggunaan *vanilla* PHP di *template blade*.

[back to top](#content1)

=================================================================

## Table Naming Guidelines
### Content2

## Database Naming

1. **Use Descriptive Names:** Choose a name for your database that clearly represents its purpose or the application it serves. Avoid vague or generic names.

2. **Use Lowercase Letters:** Use lowercase letters for database names to ensure consistency and reduce the risk of errors.

3. **Use Underscores or Hyphens:** To improve readability, you can separate words in the database name with underscores (_) or hyphens (-). For example, "my_database" or "my-database."

[back to top](#content2)

## Table Naming

1. **Use Singular Nouns:** Name your tables using singular nouns rather than plural nouns. For example, "user" instead of "users."

2. **Be Consistent:** Maintain consistency in table names throughout your database. Use a standard naming convention and stick to it.

3. **Use Underscores:** To separate words in table names, consider using underscores, making them more readable. For example, "user_profile" or "order_history."

4. **Include Relationships:** When designing tables that have relationships with other tables, use names that reflect the relationship. For example, "user_id" is a common foreign key for a user-related table.

5. **Avoid Reserved Words:** Do not use database or SQL reserved words as table names to prevent conflicts and errors. Each database management system may have a list of reserved words.

6. **Use Snake Case:** If you choose to use word separation, use snake_case (e.g., "order_details") rather than camelCase or PascalCase.

[back to top](#content2)

## Foreign Key

1. Because in kalbe we integrate with many server/database then foreign key is not neccessary

[back to top](#content2)

## View Naming

1. Use prefix "vw" for any view created, example "vw_struktur_sales_mkt"

[back to top](#content2)

## Stored Procedure Naming

1. Use prefix "sp" for any stored procedure created, example "sp_generate_product"

[back to top](#content2)

=================================================================

## UI/UX
### Content3

## Login Form
1. Contain Logo of application, can be on left or middle side
2. Contain placeholder for username and password
3. Login button that easy to read
4. Name of app that defined on login form
5. Don't forget add validation for wrong password/username

[back to top](#content3)

## Maintainable Menu and Access Menu
1. App contain a maintanable master menu and access menu
2. Access menu able to apply for *jabatan* or by *nik*

[back to top](#content3)

## Dropdown
1. Use select2 searable dropdown

[back to top](#content3)

## Template
1. It is recommended to use the available templates

[back to top](#content3)

