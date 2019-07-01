# SplObjectStorage的使用

## 简介
```
The SplObjectStorage class provides a map from objects to data or, by ignoring data, an object set. This dual purpose can be useful in many cases involving the need to uniquely identify objects.
```
该类提供了一个由对象到数据的一个映射的功能，或通过忽略数据来提供一个对象的集合。在很多涉及到需要对对象进行唯一标识的情况中，这两个功能是很多用的。

## 特性
继承了`Countable`，`Iterator`，`Serializable`，`ArrayAccess` 四个接口，提供了统计功能，迭代功能，序列化功能，数组访问功能。

```php
SplObjectStorage implements Countable , Iterator , Serializable , ArrayAccess
{
// Adds all objects from another storage
public addAll ( SplObjectStorage $storage ) : void
// Checks if the storage contains a specific object
public contains ( object $object ) : bool
// Adds an object in the storage
public attach ( object $object [, mixed $data = NULL ] ) : void
// Removes an object in the storage
public detach ( object $object ) : void
// Removes objects contained in another storage from the current storage
public removeAll ( SplObjectStorage $storage ) : void
// Removes all objects except for those contained in another storage from the current storage
public removeAllExcept ( SplObjectStorage $storage ) : void
// Calculate a unique identifier for the contained objects
public getHash ( object $object ) : string
// Sets the data associated with the current iterator entry
public setInfo ( mixed $data ) : void
// Returns the data associated with the current iterator entry
public getInfo ( void ) : mixed

// Countable接口
public count ( void ) : int

// Iterator接口
public rewind ( void ) : void
public valid ( void ) : bool
public key ( void ) : int
public current ( void ) : object
public next ( void ) : void

// Serializable接口
public serialize ( void ) : string
public unserialize ( string $serialized ) : void

// ArrayAccess接口
public offsetExists ( object $object ) : bool
public offsetGet ( object $object ) : mixed
public offsetSet ( object $object [, mixed $data = NULL ] ) : void
public offsetUnset ( object $object ) : void
}
```
## 使用场景
类似于提供了一个容器，将对象attach或者detach，并且提供了数组访问功能（可以像数组那样操作对象啦），迭代能力（方便遍历），还有count统计能力和序列化能力（都提供了数组访问和遍历的能力，提供这个也是补充）。
类比：有两个人（SplObjectStorage1, SplObjectStorage2）和三堆不同的水果（object1，object2，object3），每个人喜欢吃的水果会有所差别，选取的水果会不一样，统计每个人选取的水果种类（特性1：通过忽略数据来提供一个对象的集合）和数量（特性2：一个由对象到数据的一个映射的功能）。

### 举例
可以重点关注下参考2

1.
```php
<?php
// As an object set
$s = new SplObjectStorage();
$o1 = new StdClass;
$o2 = new StdClass;
$o3 = new StdClass;
$s->attach($o1);
$s->attach($o2);
var_dump($s->contains($o1));
var_dump($s->contains($o2));
var_dump($s->contains($o3));
$s->detach($o2);
var_dump($s->contains($o1));
var_dump($s->contains($o2));
var_dump($s->contains($o3));
?>
```

2.
```php
<?php
// As a map from objects to data
$s = new SplObjectStorage();
$o1 = new StdClass;
$o2 = new StdClass;
$o3 = new StdClass;
$s[$o1] = "data for object 1";
$s[$o2] = array(1,2,3);
if (isset($s[$o2])) {
  var_dump($s[$o2]);
}
?>
```

3.
```php
<?php
class Subject implements \SplSubject
{
    private $observers;
    
    public function __construct()
    {
        $this->observers = new \SplObjectStorage;
    }
    public function attach(\SplObserver $observer)
    {
        $this->observers->attach($observer);
    }
    public function detach(\SplObserver $observer)
    {
        $this->observers->detach($observer);
    }
    public function notify()
    {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }
}
```
?>

## 参考
1. [https://www.php.net/manual/en/class.splobjectstorage.php](https://www.php.net/manual/en/class.splobjectstorage.php)
2. [https://hotexamples.com/examples/-/SplObjectStorage/-/php-splobjectstorage-class-examples.html](https://hotexamples.com/examples/-/SplObjectStorage/-/php-splobjectstorage-class-examples.html)