# PHP 8.1 新特性详解

PHP 8.1 于2021年11月25日发布，是PHP语言的一个重大更新，引入了许多令人兴奋的新特性。

## 🎯 核心新特性

### 1. 枚举 (Enums)

PHP 8.1引入了原生枚举支持，提供了类型安全的常量集合。

```php
// 纯枚举
enum Status
{
    case PENDING;
    case RUNNING;
    case COMPLETED;
}

// 支持值的枚举
enum StatusCode: int
{
    case PENDING = 1;
    case RUNNING = 2;
    case COMPLETED = 3;
    
    public function label(): string
    {
        return match($this) {
            self::PENDING => '待处理',
            self::RUNNING => '运行中',
            self::COMPLETED => '已完成',
        };
    }
}

// 使用示例
function processTask(Status $status): void
{
    match($status) {
        Status::PENDING => echo "任务待处理",
        Status::RUNNING => echo "任务运行中",
        Status::COMPLETED => echo "任务已完成",
    };
}
```

### 2. 纤程 (Fibers)

纤程提供了轻量级的协程支持，允许在任意点暂停和恢复执行。

```php
$fiber = new Fiber(function (): void {
    $value = Fiber::suspend('fiber');
    echo "Value used to resume fiber: ", $value, PHP_EOL;
});

$value = $fiber->start();
echo "Value from fiber suspending: ", $value, PHP_EOL;

$fiber->resume('test');

// 实际应用示例
function fetchData($url) {
    $fiber = new Fiber(function() use ($url) {
        $data = Fiber::suspend($url);
        return processData($data);
    });
    
    $suspendedUrl = $fiber->start();
    $response = httpRequest($suspendedUrl);
    return $fiber->resume($response);
}
```

### 3. 只读属性 (Readonly Properties)

只读属性只能在声明时或构造函数中初始化一次。

```php
class User
{
    public readonly string $id;
    public readonly string $name;
    
    public function __construct(string $id, string $name)
    {
        $this->id = $id;
        $this->name = $name;
    }
}

$user = new User('123', 'John');
echo $user->name; // 可以读取
// $user->name = 'Jane'; // 错误：不能修改只读属性
```

### 4. 一等可调用语法 (First-class Callable Syntax)

新的语法使得创建可调用对象更加简洁。

```php
// 旧语法
$fn = 'strlen';
$fn = Closure::fromCallable('strlen');

// 新语法
$fn = strlen(...);
$fn = $obj->method(...);
$fn = $obj::staticMethod(...);

// 使用示例
$numbers = [1, 2, 3, 4, 5];
$strings = array_map(strval(...), $numbers);

class Calculator
{
    public function add($a, $b) { return $a + $b; }
}

$calc = new Calculator();
$addFunction = $calc->add(...);
echo $addFunction(5, 3); // 输出: 8
```

### 5. 交集类型 (Intersection Types)

允许指定一个值必须同时满足多个类型约束。

```php
interface Loggable
{
    public function log(): void;
}

interface Cacheable
{
    public function cache(): void;
}

class Service implements Loggable, Cacheable
{
    public function log(): void { /* ... */ }
    public function cache(): void { /* ... */ }
}

// 交集类型
function process(Loggable&Cacheable $service): void
{
    $service->log();
    $service->cache();
}

process(new Service()); // 正确
```

### 6. never 返回类型

表示函数永远不会正常返回。

```php
function redirect(string $url): never
{
    header('Location: ' . $url);
    exit();
}

function throwException(): never
{
    throw new Exception('Something went wrong');
}
```

## 🔧 其他重要改进

### 新的数组函数

```php
// array_is_list() - 检查数组是否为列表
$list = [1, 2, 3];
$map = ['a' => 1, 'b' => 2];

var_dump(array_is_list($list)); // true
var_dump(array_is_list($map));  // false
```

### 新的字符串函数

```php
// str_contains() 的补充函数
str_starts_with('Hello World', 'Hello'); // true
str_ends_with('Hello World', 'World');   // true
```

### 新的文件系统函数

```php
// fsync() 和 fdatasync()
$file = fopen('data.txt', 'w');
fwrite($file, 'Hello World');
fsync($file);  // 强制写入磁盘
fclose($file);
```

## 📈 性能改进

- **JIT编译器优化**: 改进了即时编译器的性能
- **继承缓存**: 减少了类继承的开销
- **更快的类解析**: 优化了类和函数的解析速度
- **内存使用优化**: 减少了内存占用

## ⚠️ 向后不兼容的变更

### 1. 资源到对象的迁移

许多资源类型已转换为对象：

```php
// PHP 8.0 及之前
$stream = fopen('file.txt', 'r');
var_dump(is_resource($stream)); // true

// PHP 8.1
$stream = fopen('file.txt', 'r');
var_dump($stream instanceof \GdImage); // 对于GD资源
```

### 2. 默认错误报告级别变更

```php
// PHP 8.1 默认包含 E_ALL
error_reporting(E_ALL); // 新的默认值
```

## 🚀 升级建议

1. **测试枚举兼容性**: 确保现有代码不与新的enum关键字冲突
2. **检查只读属性**: 评估在现有类中使用只读属性的机会
3. **性能测试**: 利用新的性能改进进行基准测试
4. **代码审查**: 使用新的语法特性改进代码质量

## 📚 实际应用示例

### 状态机实现

```php
enum OrderStatus: string
{
    case PENDING = 'pending';
    case CONFIRMED = 'confirmed';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';
    case CANCELLED = 'cancelled';
    
    public function canTransitionTo(self $status): bool
    {
        return match([$this, $status]) {
            [self::PENDING, self::CONFIRMED] => true,
            [self::PENDING, self::CANCELLED] => true,
            [self::CONFIRMED, self::SHIPPED] => true,
            [self::SHIPPED, self::DELIVERED] => true,
            default => false,
        };
    }
}
```

### 不可变数据对象

```php
class Point
{
    public readonly float $x;
    public readonly float $y;
    
    public function __construct(float $x, float $y)
    {
        $this->x = $x;
        $this->y = $y;
    }
    
    public function distance(Point $other): float
    {
        return sqrt(
            pow($this->x - $other->x, 2) + 
            pow($this->y - $other->y, 2)
        );
    }
}
```

---

*PHP 8.1 为现代PHP开发带来了强大的新工具，特别是枚举和纤程为构建更安全、更高效的应用程序提供了基础。*
