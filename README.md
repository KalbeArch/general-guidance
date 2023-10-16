[general-guidance v1.0](#general-guidance)

=================================================================

- [Code Writing Guidelines](#code-writing-guidelines)
- [Table Naming Guidelines](#table-naming-guidelines)
- [UI/UX](#ui-ux)
- [Version Control](#version-control)
- [Scheduler/Stored Procedure Documentation](#sch-docs)
- [Merging Documentation](#merging-docs)
- [Penetration Test Guidance](#pentest-docs)

=================================================================

## Code Writing Guidelines

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

[back to top](#general-guidance)

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

[back to top](#general-guidance)

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

[back to top](#general-guidance)

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

[back to top](#general-guidance)

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

[back to top](#general-guidance)

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

[back to top](#general-guidance)

=================================================================

## Table Naming Guidelines

## Database Naming

1. **Use Descriptive Names:** Choose a name for your database that clearly represents its purpose or the application it serves. Avoid vague or generic names.

2. **Use Lowercase Letters:** Use lowercase letters for database names to ensure consistency and reduce the risk of errors.

3. **Use Underscores or Hyphens:** To improve readability, you can separate words in the database name with underscores (_) or hyphens (-). For example, "my_database" or "my-database."

[back to top](#general-guidance)

## Table Naming

1. **Use Singular Nouns:** Name your tables using singular nouns rather than plural nouns. For example, "user" instead of "users."

2. **Be Consistent:** Maintain consistency in table names throughout your database. Use a standard naming convention and stick to it.

3. **Use Underscores:** To separate words in table names, consider using underscores, making them more readable. For example, "user_profile" or "order_history."

4. **Include Relationships:** When designing tables that have relationships with other tables, use names that reflect the relationship. For example, "user_id" is a common foreign key for a user-related table.

5. **Avoid Reserved Words:** Do not use database or SQL reserved words as table names to prevent conflicts and errors. Each database management system may have a list of reserved words.

6. **Use Snake Case:** If you choose to use word separation, use snake_case (e.g., "order_details") rather than camelCase or PascalCase.

[back to top](#general-guidance)

## Foreign Key

1. Because in kalbe we integrate with many server/database then foreign key is not neccessary

[back to top](#general-guidance)

## View Naming

1. Use prefix "vw" for any view created, example "vw_struktur_sales_mkt"

[back to top](#general-guidance)

## Stored Procedure Naming

1. Use prefix "sp" for any stored procedure created, example "sp_generate_product"

[back to top](#general-guidance)

=================================================================

## UI/UX

## Login Form
1. Contain Logo of application, can be on left or middle side
2. Contain placeholder for username and password
3. Login button that easy to read
4. Name of app that defined on login form
5. Don't forget add validation for wrong password/username

[back to top](#general-guidance)

## Maintainable Menu and Access Menu
1. App contain a maintanable master menu and access menu
2. Access menu able to apply for *jabatan* or by *nik*

[back to top](#general-guidance)

## Dropdown
1. Use select2 searable dropdown

[back to top](#general-guidance)

## Template
1. It is recommended to use the available templates

[back to top](#general-guidance)

## Report
1. Table visual able to use datatable
2. Please Provide filter, searchable table value

## Any Delete Action
1. Don't forget to add confirmation modal pop-up if there is any delete action

[back to top](#general-guidance)

=================================================================

## Version Control

## Add your repo on IT Pharma Repository
1. Repo link: [http://10.105.1.135/Home/LogOn]

## Commit message
1. add description: “Add,” “Fix,” or “Update”
2. create branch if there is a new version of app

[back to top](#general-guidance)

=================================================================

## Scheduler/Stored Procedure Documentation
### Content5

## You can read existing API/Stored Procedure Documentation
Please add yours too!
1. link: [https://docs.google.com/spreadsheets/d/1tRt-RaqfTPyfBSZlUtltAmiLBwVBudMsPrxu2GLF_Gw/edit#gid=0]

[back to top](#general-guidance)

=================================================================

## Merging Documentation

## If your app using the NIK Functional please list your table

1. link: [https://docs.google.com/spreadsheets/d/14MLIRjkkypdKFSrsruDLtoMSWSxzw4oBexVlD5mCNnY/edit#gid=0]
   
[back to top](#general-guidance)

=================================================================

## Penetration Test Guidance

## If your app is internet connection, please refer to this documentation

1. link: [https://kalbecorp-my.sharepoint.com/:x:/p/pramitha_widyasari/EQr_TK9IWx1Ng7KWxfLlebcBli77pZ40ekbIkFkcHcR8bA?e=lqogcb]
   
[back to top](#general-guidance)
