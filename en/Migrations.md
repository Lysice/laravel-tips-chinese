## Migrations

⬆️ [Go to top](#laravel-tips) ⬅️ [Previous (Models Relations)](#models-relations) ➡️ [Next (Views)](#views)

1. [Unsigned Integer](#unsigned-integer)

2. [Order of Migrations](#order-of-migrations)

3. [Migration fields with timezones](#migration-fields-with-timezones)

4. [Database migrations column types](#database-migrations-column-types)

5. [Default Timestamp](#default-timestamp)

6. [Migration Status](#migration-status)

7. [Create Migration with Spaces](#create-migration-with-spaces)

8. [Create Column after Another Column](#create-column-after-another-column)

9. [Make migration for existing table](#make-migration-for-existing-table)

10. [Output SQL before running migrations](#output-sql-before-running-migrations)

11. [Anonymous Migrations](#anonymous-migrations)

12. [You can add "comment" about a column inside your migrations](#you-can-add-comment-about-a-column-inside-your-migrations)
13. [Checking For Table / Column Existence](#checking-for-table-column-existence)


### Unsigned Integer

For foreign key migrations instead of `integer()` use `unsignedInteger()` type or `integer()->unsigned()`, otherwise you may get SQL errors.

```php
Schema::create('employees', function (Blueprint $table) {
    $table->unsignedInteger('company_id');
    $table->foreign('company_id')->references('id')->on('companies');
    // ...
});
```

You can also use `unsignedBigInteger()` if that other column is `bigInteger()` type.

```php
Schema::create('employees', function (Blueprint $table) {
    $table->unsignedBigInteger('company_id');
});
```

### Order of Migrations

If you want to change the order of DB migrations, just rename the file's timestamp, like from `2018_08_04_070443_create_posts_table.php` to`2018_07_04_070443_create_posts_table.php` (changed from `2018_08_04` to `2018_07_04`).

They run in alphabetical order.

### Migration fields with timezones

Did you know that in migrations there's not only `timestamps()` but also `timestampsTz()`, for the timezone?

```php
Schema::create('employees', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email');
    $table->timestampsTz();
});
```

Also, there are columns `dateTimeTz()`, `timeTz()`, `timestampTz()`, `softDeletesTz()`.

### Database migrations column types

There are interesting column types for migrations, here are a few examples.

```php
$table->geometry('positions');
$table->ipAddress('visitor');
$table->macAddress('device');
$table->point('position');
$table->uuid('id');
```

See all column types on the [official documentation](https://laravel.com/docs/master/migrations#creating-columns).

### Default Timestamp

While creating migrations, you can use `timestamp()` column type with option
`useCurrent()` and `useCurrentOnUpdate()`, it will set `CURRENT_TIMESTAMP` as default value.

```php
$table->timestamp('created_at')->useCurrent();
$table->timestamp('updated_at')->useCurrentOnUpdate();
```

### Migration Status

If you want to check what migrations are executed or not yet, no need to look at the database "migrations" table, you can launch `php artisan migrate:status` command.

Example result:

```
+------+------------------------------------------------+-------+
| Ran? | Migration                                      | Batch |
+------+------------------------------------------------+-------+
| Yes  | 2014_10_12_000000_create_users_table           | 1     |
| Yes  | 2014_10_12_100000_create_password_resets_table | 1     |
| No   | 2019_08_19_000000_create_failed_jobs_table     |       |
+------+------------------------------------------------+-------+
```

### Create Migration with Spaces

When typing `make:migration` command, you don't necessarily have to use underscore `_` symbol between parts, like `create_transactions_table`. You can put the name into quotes and then use spaces instead of underscores.

```php
// This works
php artisan make:migration create_transactions_table

// But this works too
php artisan make:migration "create transactions table"
```

Source: [Steve O on Twitter](https://twitter.com/stephenoldham/status/1353647972991578120)

### Create Column after Another Column

_Notice: Only MySQL_

If you're adding a new column to the existing table, it doesn't necessarily have to become the last in the list. You can specify after which column it should be created:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->after('email');
});
```

If you're adding a new column to the existing table, it doesn't necessarily have to become the last in the list. You can specify before which column it should be created:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->before('created_at');
});
```

If you want your column to be the first in your table , then use the first method.

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('uuid')->first();
});
```

### Make migration for existing table

If you make a migration for existing table, and you want Laravel to generate the Schema::table() for you, then add "_in_xxxxx_table" or "_to_xxxxx_table" at the end, or specify "--table" parameter.
`php artisan change_fields_products_table` generates empty class

```php
class ChangeFieldsProductsTable extends Migration
{
    public function up()
    {
        //
    }
}
```

But add `in_xxxxx_table` `php artisan make:migration change_fields_in_products_table` and it generates class with `Schemma::table()` pre-fileed

```php
class ChangeFieldsProductsTable extends Migration
{
    public function up()
    {
        Schema::table('products', function (Blueprint $table) {
            //
        })
    };
}
```

Also you can specify `--table` parameter `php artisan make:migration whatever_you_want --table=products`

```php
class WhateverYouWant extends Migration
{
    public function up()
    {
        Schema::table('products', function (Blueprint $table) {
            //
        })
    };
}
```

### Output SQL before running migrations

When typing `migrate --pretend` command, you get the SQL query that will be executed in the terminal. It's an interesting way to debug SQL if necessary.

```php
// Artisan command
php artisan migrate --pretend
```

Tip given by [@zarpelon](https://github.com/zarpelon)

### Anonymous Migrations

The Laravel team released Laravel 8.37 with anonymous migration support, which solves a GitHub issue with migration class name collisions. The core of the problem is that if multiple migrations have the same class name, it'll cause issues when trying to recreate the database from scratch.
Here's an example from the [pull request](https://github.com/laravel/framework/pull/36906) tests:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(
    {
        Schema::table('people', function (Blueprint $table) {
            $table->string('first_name')->nullable();
        });
    }

    public function down()
    {
        Schema::table('people', function (Blueprint $table) {
            $table->dropColumn('first_name');
        });
    }
};
```

Tip given by [@nicksdot](https://twitter.com/nicksdot/status/1432340806275198978)

### You can add "comment" about a column inside your migrations

You can add "comment" about a column inside your migrations and provide useful information.<br>
If database is managed by someone other than developers, they can look at comments in Table structure before performing any operations.

```php
$table->unsignedInteger('interval')
    ->index()
    ->comment('This column is used for indexing.')   
```

Tip given by [@Zubairmohsin33](https://twitter.com/Zubairmohsin33/status/1442345998790107137)

### Checking For Table / Column Existence

You may check for the existence of a table or column using the hasTable and hasColumn methods:

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}
if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}
```

Tip given by [@dipeshsukhia](https://github.com/dipeshsukhia)

https://twitter.com/alexjgarrett/status/1465674086022107137)