# LARAVEL VERSIONIA

[![Build Status](https://travis-ci.org/jumilla/laravel-versionia.svg)](https://travis-ci.org/jumilla/laravel-versionia)
[![Quality Score](https://img.shields.io/scrutinizer/g/jumilla/laravel-versionia.svg?style=flat)](https://scrutinizer-ci.com/g/jumilla/laravel-versionia)
[![Code Coverage](https://scrutinizer-ci.com/g/jumilla/laravel-versionia/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/jumilla/laravel-versionia/)
[![Latest Stable Version](https://poser.pugx.org/jumilla/laravel-versionia/v/stable.svg)](https://packagist.org/packages/jumilla/laravel-versionia)
[![Total Downloads](https://poser.pugx.org/jumilla/laravel-versionia/d/total.svg)](https://packagist.org/packages/jumilla/laravel-versionia)
[![Software License](https://poser.pugx.org/jumilla/laravel-versionia/license.svg)](https://packagist.org/packages/jumilla/laravel-versionia)

[日本語ドキュメント - Japanese](readme-ja.md)

Version based database migration system for Laravel 5.

Laravel Versionia is version based database migration system.
It can be used in Laravel 5 and Lumen 5.

## Concepts

The feature as "migration" loads standards into Laravel 4 and 5 for management of a database (RDB) schema.

Migration is the mechanism that making of a schema and change are being managed by time series.
A PHP class for the mounting of "seed" as which data is defined an early stage of a data base and the artisan command are also offered.

Versionia makes standard migration easier to use.

- Add the programmer specified "version" to migration of Laravel 5.
- As `Seeder` class of Laravel 5 can be distinguished under the name, more than one seed is changed easily.
- It's offered by a service provider along architecture of Laravel 5.
- Migration and seed classes can be arranged under the `app` directory.

A service provider is employed as Laravel 5 application and the mechanism that a Composer package offers the function.
The definition which are routing and an event listener by a service provider is given, but migrations and seeds can be defined now here.

## Installation method

### [A] Include Laravel Extension (Laravel).

When using Laravel 5, it's recommendation here.

Use [Composer](http://getcomposer.org).

```sh
composer require laravel-plus/extension
```

Next `LaravelPlus\Extension\ServiceProvider::class` is added to a `provider` in `config/app.php`.

```php
    'providers' => [
        ...

        LaravelPlus\Extension\ServiceProvider::class,
    ],
```

Please read the explanation of [Laravel Extension](https://github.com/jumilla/laravel-extension) for more information.

### [B] Include Versionia (Laravel)

Use [Composer](http://getcomposer.org).

```sh
composer require jumilla/laravel-versionia
```

Next `Jumilla\Versionia\Laravel\ServiceProvider::class` is added to a `provider` in `config/app.php`.

```php
    'providers' => [
        ...

        Jumilla\Versionia\Laravel\ServiceProvider::class,
    ],
```

### [C] Include Versionia (Lumen)

When using Lumen, it's recommendation here.

Use [Composer](http://getcomposer.org).

```sh
composer require jumilla/laravel-versionia
```

Next the next code is added to `boostrap/app.php`.

```php
$app->register(Jumilla\Versionia\Laravel\ServiceProvider::class);
```

## Migration version definition

There was Naming Rule in the file name of migration class so far, and an order of migration had been decided by the file generation date and time when you were embedded in the file name.
A group and the version are given specifically every migration class and it's defined by the `DatabaseServiceProvider` class in Versionia.

```php
<?php

namespace App\Providers;

use Jumilla\Versionia\Laravel\Support\DatabaseServiceProvider as ServiceProvider;
use App\Database\Migrations;
use App\Database\Seeds;

class DatabaseServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->migrations('framework', [
            '1.0' => Migrations\Framework_1_0::class,
        ]);

        $this->migrations('app', [
            '1.0' => Migrations\App_1_0::class,
        ]);

        // ... seed definition ...
    }
}
```

### Registration of DatabaseServiceProvider

When making a service provider newly, please register.
When using [Laravel Extension](https://github.com/jumilla/laravel-extension) it's included already, so addition is unnecessary.

#### Laravel

`App\Providers\DatabaseServiceProvider::class` is added to `app\config.php`.

```php
    'providers' => [
        ...
        App\Providers\ConfigServiceProvider::class,
        App\Providers\DatabaseServiceProvider::class,
        App\Providers\EventServiceProvider::class,
        ...
    ],
```

#### Lumen

The next code is added to `bootstrap\app.php`.

```php
$app->register(App\Providers\DatabaseServiceProvider::class);
```

### Version Number

Versionia is using PHP standard function `version_compare()` for comparison of a version number.
Please designate a character string of a dot end as a version number.

### Migration class

The one generated by `make:migration` of Laravel 5 standard can use migration class just as it is.

When arranging in `app\Database\Migrations` directory of recommendation, please add `namespace App\Database\Migrations`.

The next code is a sample of migration definition.

```php
<?php

namespace App\Database\Migrations;

use Jumilla\Versionia\Laravel\Support\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class App_1_0 extends Migration
{
    /**
     * Migrate the database to forward.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title');
            $table->text('content');
            $table->json('properties');
            $table->timestamps();
        });
    }

    /**
     * Migrate the database to backword.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

## Seed class

A seed defines `test`, `staging`, `production` by the next sample code.
The 2nd argument of method `seeds()` designates `test` by designation of a default seed.

```php
<?php

namespace App\Providers;

use Jumilla\Versionia\Laravel\Support\DatabaseServiceProvider as ServiceProvider;
use App\Database\Migrations;
use App\Database\Seeds;

class DatabaseServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
    	//... migration definition ...

        $this->seeds([
            'test' => Seeds\Test::class,
            'staging' => Seeds\Staging::class,
            'production' => Seeds\Production::class,
        ], 'test');
    }
}
```

A seed class is described as follows.

```php
<?php

namespace App\Database\Seeds;

use Jumilla\Versionia\Laravel\Support\Seeder;
use Carbon\Carbon;

class Staging extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $now = Carbon::now();

        app('db')->table('posts')->truncate();

        app('db')->table('posts')->insert([
            'title' => 'Sample post',
            'content' => 'Hello laravel world.',
            'properties' => json_encode([
                'author' => 'Seeder',
            ], JSON_PRETTY_PRINT),
            'created_at' => $now,
            'updated_at' => $now,
        ]);
    }
}
```

## Commands

### `database:status`

Migration and seed definition, installation state are displayed.

```sh
php artisan database:status
```

### `database:upgrade`

Run migration method `up()` of all groups, then up-to-date。

```sh
php artisan database:upgrade
```

It's possible to make them seed after migration.

```sh
php artisan database:upgrade --seed <seed>
```

### `database:clean`

Run migation method `down()` of all groups, then clean state.

```sh
php artisan database:clean
```

### `database:refresh`

Migration of all groups is redone.

It's same run `database:clean` and `database:upgrade`.

```sh
php artisan database:refresh
```

It's possible to run seed after migration.

```sh
php artisan database:refresh --seed <seed>
```

### `database:rollback`

The version of the designation group is returned one.

```sh
php artisan database:rollback <group>
```

When `--all` option specified, remove all version of group.

```sh
php artisan database:rollback <group> --all
```

### `database:again`

Re-run latest migration version of group.

It's same effect as run `database:rollback <group>` and `database:upgrade`.

```sh
php artisan database:again <group>
```

It's possible to run seed after migration.

```sh
php artisan database:again <group> --seed <seed>
```

### `database:seed`

Run specified seed.

```sh
php artisan database:seed <seed>
```

When omitting `<seed>`, run default seed.

```sh
php artisan database:seed
```

## Copyright

古川 文生 / Fumio Furukawa (fumio@jumilla.me)

## License

MIT
