<p align="center">
    <img src="https://matti.suoraniemi.com/storage/forur3.png" width="400">
</p>

<p align="center">
    <a href="https://packagist.org/packages/sirmathays/convenient-laravel-macros">
        <img src="https://img.shields.io/packagist/dt/sirmathays/convenient-laravel-macros" alt="Total Downloads">
    </a>
    <a href="https://packagist.org/packages/sirmathays/convenient-laravel-macros">
        <img src="https://img.shields.io/packagist/v/sirmathays/convenient-laravel-macros" alt="Latest Stable Version">
    </a>
    <a href="https://packagist.org/packages/sirmathays/convenient-laravel-macros">
        <img src="https://img.shields.io/packagist/l/sirmathays/convenient-laravel-macros" alt="License">
    </a>
</p>

This package provides some additional, convenient macros for you to use with your Laravel project.

## Installation

Install the package with Composer:

    composer require sirmathays/convenient-laravel-macros

The package registers itself automatically.

## Macros

Here's a brief documentation on the macros the package provides.

- `Illuminate\Database\Eloquent\Builder`
  - [`selectKey`](#illuminatedatabaseeloquentbuilderselectkey)
  - [`whereLike` & `orWhereLike`](#illuminatedatabaseeloquentbuilderwherelike--orwherelike)
- `Illuminate\Database\Query\Builder`
  - [`selectRawArr`](#illuminatedatabasequerybuilderselectrawarr)
- `Illuminate\Support\Collection`
  - [`mergeMany`](#illuminatesupportcollectionmergemany)
  - [`pluckMany`](#illuminatesupportcollectionpluckmany)
  - [`whereExtends`](#illuminatesupportcollectionwhereextends)
  - [`whereImplements`](#illuminatesupportcollectionwhereimplements)
  - [`whereUses`](#illuminatesupportcollectionwhereuses)
- `Illuminate\Support\Arr`
  - [`combine`](#illuminatesupportarrcombine)
  - [`fillKeys`](#illuminatesupportarrfillkeys)
  - [`join`](#illuminatesupportarrjoin)
  - [`zip`](#illuminatesupportarrzip)
  - [`unzip`](#illuminatesupportarrunzip)
- `Illuminate\Support\Str`
  - [`wrap`](#illuminatesupportstrwrap)
- `Illuminate\Support\Stringable`
  - [`wrap`](#illuminatesupportstringablewrap)
- `Carbon\CarbonPeriod`
  - [`collect`](#carboncarbonperiodcollect)

### `Illuminate\Database\Eloquent\Builder::selectKey`

Select the key of the model in the query (uses Model's `getKey` method).

```php
$query = User::query()->selectKey();

$query->toSql() // "select `id` from `users`"
```

### `Illuminate\Database\Eloquent\Builder::whereLike` & `orWhereLike`

```php
$query = User::query()
    ->whereLike('name', 'Matti Suo', 'right')
    ->orWhereLike('name', 'ranie');
    ->orWhereLike('name', 'mi', 'left');

$query->toSql(); // "select * from `users` where (`name` LIKE ?) or (`name` LIKE ?) or (`name` LIKE ?)"
// First ? being "Matti Suo%", second "%ranie%" and third "%mi"

$query = User::query()->whereLike('games.name', 'Apex Leg', 'right');

$query->toSql(); 
// select * from `users` where (exists (select * from `games` where `users`.`id` = `games`.`user_id` and `name` LIKE ?))
// ? being "Apex Leg%"
```

### `Illuminate\Database\Query\Builder::selectRawArr`

Add raw select statements as an array, instead of as a one ugly string (`selectRaw`).

```php
$query = User::query()->selectRawArr([
    'concat(`id`, "-", `name`) as id_name'
    'concat(`email`, "-", `name`) as email_name'
]);

$query->first()->toArray() // ["id_name" => "1-Matti", "email_name" => "matti@suoraniemi.com-Matti"]

// VS

$query = User::query()->selectRaw('concat(`id`, "-", `name`) as id_name, concat(`email`, "-", `name`) as email_name');
```

### `Illuminate\Support\Collection::mergeMany`

Merge multiple arrays/collections to the collection in one go.

```php
$data = new Collection([1,2,3]);

$data->mergeMany([4], [5], [6]); // [1, 2, 3, 4, 5, 6]
```

### `Illuminate\Support\Collection::pluckMany`

Pluck several keys from the collection items.

```php
$data = User::query()->get();

$data->pluckMany('id', 'name')->toArray(); // [["id" => 1, "name" => "Matti Suoraniemi"]]
```

### `Illuminate\Support\Collection::whereExtends`

Filter classes and/or objects that extend the given class.

```php
use Illuminate\Database\Eloquent\Model;

$data = new Collection([
    \App\Models\User::class,
    \App\Models\Game::class,
    \App\Models\Console::class,
    \App\Models\Hobby::class,
]);

$data->whereExtends(Model::class)->count(); // 4
```

### `Illuminate\Support\Collection::whereImplements`

Filter classes and/or objects that implement the given interface.

```php
use App\Contracts\PlayableOnConsole;

$data = new Collection([
    \App\Models\User::class,
    \App\Models\Game::class,
    \App\Models\Console::class,
    \App\Models\Hobby::class,
]);

$data->whereImplements(PlayableOnConsole::class)->toArray(); // ["App\Models\Game"]
```

### `Illuminate\Support\Collection::whereUses`

Filter classes and/or objects that use the given trait.

```php
use Illuminate\Notifications\Notifiable;

$data = new Collection([
    \App\Models\User::class,
    \App\Models\Game::class,
    \App\Models\Console::class,
    \App\Models\Hobby::class,
]);

$data->whereUses(Notifiable::class)->toArray(); // ["App\Models\User"]
```

### `Illuminate\Support\Arr::associate`

Converts an array into a fully associative array by converting any values with an integer key
to the value being the key filled with the given fill value. Values that have a string key already
won't be touched.

```php
Arr::associate(['foo']) // ["foo" => null]
Arr::associate(['foo', 'bar' => []], []) // ["foo" => [], "bar" => []]
Arr::associate(['foo', 'bar' => []], fn () => Arr::random(['foo', 'bar'])) // ["foo" => "foo", "bar" => []]
Arr::associate([fn () => Str::reverse('foo'), 'bar' => []]) // ["oof" => null, "bar" => []]
```

### `Illuminate\Support\Arr::combine`

Similar to `array_combine`, but allows to have more keys than values. Keys without value will be set
as null.

```php
Arr::combine(['foo', 'zoo'], ["bar", "gar"]) // ["foo" => "bar", "zoo" => "gar"]
Arr::combine(['foo', 'zoo'], ["bar"]) // ["foo" => "bar", "zoo" => null]
```

### `Illuminate\Support\Arr::fillKeys`

Fills given keys with given value. You can also set that only keys that already exist in the array
can become filled. In other words, if the key `foo` is to be filled with value `bar`, but the key
`foo` doesn't already exist in the array, the array will remain unchanged.

```php
$array = ['foo' => 'bar', 'zoo' => 'gar'];

Arr::fillKeys($array, ['foo', 'zoo'], null) // ["foo" => null, "zoo" => null]
Arr::fillKeys($array, ['foo', 'zoo', 'boo'], null) // ["foo" => null, "zoo" => null, "boo" => null]
Arr::fillKeys($array, ['foo', 'zoo', 'boo'], null, true) // ["foo" => null, "zoo" => null]
```

### `Illuminate\Support\Arr::join`

Collection's nice join method brought to Arr.

```php
Arr::join(['foo', 'bar', 'zoo'], ', ', ' and ') // "foo, bar and zoo"
```

### `Illuminate\Support\Arr::zip`

Zips the key and value together with the given zipper.

```php
Arr::zip(['foo' => 'bar', 'zoo' => 'gar'], ':') // ["foo:bar", "zoo:gar"]
```

### `Illuminate\Support\Arr::unzip`

Unzips keys to key and value with the given zipper.

```php
Arr::unzip(['foo:bar', 'zoo:gar'], ':') // ["foo" => "bar", "zoo" => "gar"]
```

### `Illuminate\Support\Str::wrap`

Wraps the string with given character(s).

```php
Str::wrap('foo', ':') // ":foo:"
Str::wrap('bar', '<', '>') // "<bar>"
Str::wrap('!zoo', '!') // "!zoo!"
```

### `Illuminate\Support\Stringable::wrap`

Wraps the string with given character(s).

```php
(string) Str::of('foo')->upper()->wrap(':') // ":FOO:"
(string) Str::of('bar')->upper()->wrap('<', '>') // "<BAR>"
(string) Str::of('!zoo')->upper()->wrap('!') // "!ZOO!"
```

### `Carbon\CarbonPeriod::collect`

```php
$dates = CarbonPeriod::between('yesterday', 'today')->collect();

$dates->first()->toDateTimeString() // "2022-06-14 00:00:00"
```

## Extending the MacroServiceProvider

You can make your own pretty MacroServiceProviders by extending the MacroServiceProvider class
provided by this package!

Here's an example:

```php
<?php

namespace App\Providers;

use SirMathays\Convenience\Foundation\MacroServiceProvider as ServiceProvider;

class MacroServiceProvider extends ServiceProvider
{
    protected static array $macros = [
        \Illuminate\Support\Collection::class => [
            \App\Macros\MyOwnCollectionMacro::class
        ]
    ];
}
```

The macro class referenced in the example above would look something like:

```php
<?php

namespace App\Macros;

class MyOwnCollectionMacro
{
    public function __invoke(): \Closure
    {
        return function () {
            // Something cool worth getting macroed happens here...
            return $this;
        }
    }
}
```

## License

Convenient Laravel Commands is open-sourced software licensed under the [MIT license](LICENSE.md).
