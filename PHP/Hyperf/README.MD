
### 使用Hyperf的开发调试工具
```shell
composer require hyperf/devtool --dev
```


### 输出刚刚执行的 SQL
```php
<?php
use Hyperf\DbConnection\Db;
use Hyperf\Utils\Arr;
use App\Model\Book;

// 启用 SQL 数据记录功能
Db::enableQueryLog();

$book = Book::query()->find(1);

// 打印最后一条 SQL 相关数据
var_dump(Arr::last(Db::getQueryLog()));
```

## 查询构造器
```php
<?php
use Hyperf\DbConnection\Db;

$users = Db::select('SELECT * FROM user;');             //
$users = Db::table('user')->get();
$users = Db::table('user')->select('name', 'gender as user_gender')->get();
```

### 获取一行的值
```
Db::table('user')->first(); // sql 会自动加上 limit 1
```
返回stdClass对象

### 获取单个值
