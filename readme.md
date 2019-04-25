Laravel XunSearch
========
介绍
--------
这个包是在 [laravel/scout][laravel_scout_url] 的服务中添加拓展，使用 [XunSearch 搜索][xun_search_sdk_url] 功能。

XunSearch 的安装，具体查看 [XunSearch 的官方文档][xun_search_index]。

laravel/scout 的安装和使用，查看 [laravel/scout 的官方文档][laravel_scout_doc]。

安装
--------

使用 composer
```shell
composer require linshunwei/xunsearch-laravel
```

在配置文件中添加服务提供者（Laravel5.5 有自动添加）
```php
'providers' => [
    //...
    linshunwei\XunSearchLaravel\XunSearchServiceProvider::class,
    //...
],
```

复制配置文件到配置目录，配置文件内容不多，而且可以在 `.env` 文件中设置。手动复制或者使用命令复制：
```shell
php artisan vendor:publish --provider="Linshunwei\XunSearchLaravel\XunSearchServiceProvider"
```

修改 scout 配置文件 `config/scout.php`，让 scout 使用 XunSearch 引擎
```php
    'driver' => env('SCOUT_DRIVER', 'xunsearch'),
```

或者直接在 `.env` 文件中设置
```
SCOUT_DRIVER=xunsearch
```

修改 XunSearch 配置文件 `config/xunsearch.php`
```php
    'server_host' => env('XUNSEARCH_SERVER_HOST', '127.0.0.1'),
    'server_index_port' => env('XUNSEARCH_SERVER_INDEX_PORT', '8383'),
    'server_search_port' => env('XUNSEARCH_SERVER_SEARCH_PORT', '8384'),
    'default_charset' => env('XUNSEARCH_DEFAULT_CHARSET', 'utf-8'),
```

或者直接在 `.env` 文件中设置需要修改的内容，没有特殊情况默认即可
```
XUNSEARCH_SERVER_HOST=127.0.0.1
```

使用
--------
在 `Model` 中使用搜索功能，先引入 `Searchable` Trait，详见 [Scout 使用文档][laravel_scout_doc]。
```php
use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Blog extends Model
{
    use Searchable;
```

要使用 XunSearch， `Model` 还要实现指定接口，同时如果有需要，可以使用 `XunSearchTrait` Trait （Trait 中注册了范围检索方法 `range` 和清空所有数据方法 `cleanSearchable`）。
实现接口需要添加 `xunSearchFieldsType` 方法进行字段类型设置
```
use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;
use Linshunwei\XunSearchLaravel\Contracts\XunSearch as XunSearchContract;
use Linshunwei\XunSearchLaravel\XunSearchTrait;

class Blog extends Model implements XunSearchContract
{
    use Searchable， XunSearchTrait;
    
    public function xunSearchFieldsType()
    {
        return [
            'id' => [
                'type'=>self::XUNSEARCH_TYPE_NUMERIC,
            ],
            'title' => [
                'type'=>self::XUNSEARCH_TYPE_TITLE,
            ],
            'body' => [
                'type'=>self::XUNSEARCH_TYPE_BODY,
            ],
            'field' => [
                'tokenizer'=>self::XUNSEARCH_TOKENIZER_XLEN,
                'tokenizer_value'=>2,
            ],
            'created_at' => [
                'type'=>self::XUNSEARCH_TYPE_DATE,
                'index'=>self::XUNSEARCH_INDEX_NONE,
            ],
        ];
    }
```

设置的字段类型的具体效果，查看 [XunSearch 官方文档][xun_search_index]。

`Model` 的主键，例如 `id`，已被默认设为引擎的文档主键。
如果需要对 id 进行区间检索，把 id 的类型设为 `self::XUNSEARCH_TYPE_NUMERIC`。如果不需要对 `id` 进行检索，可以不添加 `id` 字段。

字段类型 `self::XUNSEARCH_TYPE_TITLE` 和 `self::XUNSEARCH_TYPE_BODY` 只能分别设置一次。

`XunSearchTrait` 中给 Scout 的 `Builder` 注册了 `range` 方法进行区间检索。除了 `title` 和 `body` 特殊字段， XunSearch 默认设定字段为 `string`，需要进行区间检索的字段，要设为 `numeric` 或者 `date` 才能正常检索。

例如设定 `id` 字段为 `self::XUNSEARCH_TYPE_NUMERIC`，在 `id` 大于 `20` 小于等于 `60` 的范围内搜索 `word`
```
Blog::search('word')->range('id', 20, 60)->get();
```

字段的索引 `index` 和分词器 `tokenizer` 的设置效果，在 [XunSearch 官方文档][xun_search_index] 查看。


* `XunSearchTrait` 中移除 `cleanSearchable` 方法，请使用 Scout 中的 `removeAllFromSearch` 方法。

相关链接
--------
[laravel/scout][laravel_scout_url]

[laravel/scout 文档][laravel_scout_doc]

[hightman/xs-sdk-php][xun_search_sdk_url]

[XunSearch 文档][xun_search_index]

[laravel_scout_url]: https://github.com/laravel/scout
[laravel_scout_doc]: https://laravel.com/docs/master/scout
[xun_search_sdk_url]: https://github.com/hightman/xs-sdk-php
[xun_search_index]: http://www.xunsearch.com
