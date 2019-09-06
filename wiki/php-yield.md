# 生成器（yield）

## 概念

```
The heart of a generator function is the yield keyword. In its simplest form, a yield statement looks much like a return statement, except that instead of stopping execution of the function and returning, yield instead provides a value to the code looping over the generator and pauses execution of the generator function.
```

大致的意思是：生成器函数的核心是`yield`关键字。它最简单的调用形式看起来像一个`return`申明，不同之处在于普通`return`会返回值并终止函数的执行，而`yield`会返回一个值给循环调用此生成器的代码并且只是暂停执行生成器函数。（意味着交出控制权，返还给主函数，如果主函数再次执行到这里，会继续执行生成器函数）。

## 几个关键词理解

### 遍历
对一个数组进行遍历，例如foreach对数组进行遍历，但是如果数组的长度很大，使用该方式会占用很大的内存。

```php
function range($start, $end, $step = 1) {
  $res = [];
  for ($i = $start; $i <= $end; $i += $step) {
    $res[] = $i;
  }
  return $res;
}
// 使用方法
foreach (range(0, 9) as $key => $val) {
  echo $key, ' ', $val, "\n";
}
```

### 迭代器
迭代器即实现`Iterator`接口，即可迭代，即可以使用`foreach`语句进行遍历。使用迭代器实现遍历内存损耗远小于数组遍历，foreach迭代时，参数是一个实现`Iterator`接口的迭代器对象实例）。

```php
class Xrange implements Iterator
{
    protected $start;
    protected $limit;
    protected $step;
    protected $current;

    public function __construct($start, $limit, $step = 1)
    {
        $this->start = $start;
        $this->limit = $limit;
        $this->step  = $step;
    }
    
    public function rewind()
    {
        $this->current = $this->start;
    }

    public function next()
    {
        $this->current += $this->step;
    }

    public function current()
    {
        return $this->current;
    }

    public function key()
    {
        return $this->current + 1;
    }

    public function valid()
    {
        return $this->current <= $this->limit;
    }
}
// 使用方法
foreach (new Xrange(0, 9) as $key => $val) {
    echo $key, ' ', $val, "\n";
}
```

### 生成器
相比迭代器，生成器提供了一种更简单的方式来实现简单的对象迭代，使用生成器实现(生成器的灵魂在于`yield`，生成器生成的是一个`Generator`对象实例，该对象继承了`Iterator`接口。可以看出和迭代器类似)。
下例中函数xrange返回的是一个`Generator`对象，对该对象遍历时会遵循yield的特性，第一次迭代时，执行 `$i = 0`、`$i < $limit`、`yield` 返回 `1 0`，然后中断，第二次迭代时，执行`$i += $step`、`$i < $limit`, `yield`返回`2 1`，然后中断，以此类推。。。

```php
function xrange($start, $limit, $step = 1) {
    for ($i = 0; $i < $limit; $i += $step) { 
        yield $i + 1 => $i;
    }
}

foreach (xrange(0, 9) as $key => $val) {
    printf("%d %d \n", $key, $val);
}
```

## 生成器的特性

1. 生成器为可中断函数
如上例

2. 值可传递
方法`printer()`返回的是一个`Generarot`对象，该对象有一个`send()`方法，可以传递参数。

```php
function printer()
{
    while (true) {
        printf("receive: %s\n", yield);
    }
}
$printer = printer();
$printer->send('hello');
$printer->send('world');
// 输出
receive: hello
receive: world
```

3. 双向通信
可以在两个层级间（主函数代码片段、`yield`代码片段）实现可以传递参数，也能接收结果。

```php
function gen() {
    $ret = (yield 'yield1');
    var_dump($ret);
    $ret = (yield 'yield2');
    var_dump($ret);
}

$gen = gen();
var_dump($gen->current());    // string(6) "yield1"
var_dump($gen->send('ret1')); // string(4) "ret1"   (第一个 var_dump)
                            // string(6) "yield2" (继续执行到第二个 yield，吐出了返回值)
var_dump($gen->send('ret2')); // string(4) "ret2"   (第二个 var_dump)
                            // NULL (var_dump 之后没有其他语句，所以这次 ->send() 的返回值为 null)
```

## 总结
yield关键字不仅可用于迭代数据，也因为它的双向通信，可用于协程在php语言中的实现，必须清楚的是`yield`是生成器里面的关键字，协程能够使用生成器来实现，是因为生成器可以双向通信，当然协程也可以使用其他的方式实现，例如swoole的协程实现方式。（关于协程暂时不多说，该篇文章主要介绍的是`yield`关键字，即生成器），关于更多的生成器示例可以google。

## 参考
1. [http://www.codeceo.com/article/php-principle-of-association.html](http://www.codeceo.com/article/php-principle-of-association.html)
2. [https://www.php.net/manual/en/class.iterator.php](https://www.php.net/manual/en/class.iterator.php)
3. [https://www.php.net/manual/en/language.generators.syntax.php](https://www.php.net/manual/en/language.generators.syntax.php)
4. [https://www.php.net/manual/en/class.generator.php](https://www.php.net/manual/en/class.generator.php)
5. [https://stackoverflow.com/questions/17483806/what-does-yield-mean-in-php](https://stackoverflow.com/questions/17483806/what-does-yield-mean-in-php)