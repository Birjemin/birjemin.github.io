# Pimple依赖注入容器

## 名词简介
* 匿名函数：匿名函数（Anonymous functions），也叫闭包函数（closures），允许 临时创建一个没有指定名称的函数。最经常用作回调函数（callback）参数的值。当然，也有其它应用的情况。`匿名函数目前是通过 Closure 类来实现的`。
* __invoke：当尝试以调用函数的方式调用一个对象时，__invoke() 方法会被自动调用。
* is_object：判断是否为一个对象。

## 简介
```
Pimple is a simple PHP Dependency Injection Container
```

简单点就是一个Php的依赖注入容器，主页地址：[pimple](https://pimple.symfony.com/)，什么是依赖注入呢？参见[依赖注入](http://birjemin.com/wiki/php-spl-object-storage)。可以理解为pimple是依赖注入的一种实现方式，但不局限于这种方式（Laravel的依赖注入就不是通过这种方式的，而是采用了反射的方式）。

## 源码阅读
```php
<?php
namespace Pimple;

use Pimple\Exception\ExpectedInvokableException;
use Pimple\Exception\FrozenServiceException;
use Pimple\Exception\InvalidServiceIdentifierException;
use Pimple\Exception\UnknownIdentifierException;

/**
 * Container main class.
 *
 * @author Fabien Potencier
 */
class Container implements \ArrayAccess
{
    private $values = array(); // 存储的是具体的服务（值、对象、闭包）
    private $factories; // 存储工厂方法的服务
    private $protected; // 存储受保护方法的服务
    private $frozen = array(); // 该数组中的对象不允许更改，服务一旦被第一次调用，就会被冻结（offsetGet方法中实现）,不允许更改服务（offsetSet方法中实现）。
    private $raw = array(); // 存储服务（如果服务调用过则将values的服务移动到raw中，values存储服务的执行结果）
    private $keys = array(); // 存储的是标记

    /**
     * Instantiates the container.
     *
     * Objects and parameters can be passed as argument to the constructor.
     *
     * @param array $values The parameters or objects
     */
    public function __construct(array $values = array())
    {
        // 声明SplObjectStorage对象
        $this->factories = new \SplObjectStorage(); // 工厂方法 对应extend方法
        $this->protected = new \SplObjectStorage(); // 受保护方法 对应extend方法
        // 可以通过构造函数设置服务
        foreach ($values as $key => $value) {
            $this->offsetSet($key, $value);
        }
    }

    /**
     * Sets a parameter or an object.
     *
     * Objects must be defined as Closures.
     *
     * Allowing any PHP callable leads to difficult to debug problems
     * as function names (strings) are callable (creating a function with
     * the same name as an existing parameter would break your container).
     *
     * @param string $id    The unique identifier for the parameter or object
     * @param mixed  $value The value of the parameter or a closure to define an object
     *
     * @throws FrozenServiceException Prevent override of a frozen service
     */
    public function offsetSet($id, $value)
    {
        // 如果在frozen数组中，则不允许更改
        if (isset($this->frozen[$id])) {
            throw new FrozenServiceException($id);
        }
        // 将id和value分别存储起来（value为服务）
        $this->values[$id] = $value;
        $this->keys[$id] = true;
    }

    /**
     * Gets a parameter or an object.
     *
     * @param string $id The unique identifier for the parameter or object
     *
     * @return mixed The value of the parameter or an object
     *
     * @throws UnknownIdentifierException If the identifier is not defined
     */
    public function offsetGet($id)
    {
       // 如果不在keys数组中，则认为不存在(对应offsetSet方法)
        if (!isset($this->keys[$id])) {
            throw new UnknownIdentifierException($id);
        }
        // 返回具体的值
        if (
            isset($this->raw[$id]) // row存储的是的服务，如果存在，values为其服务的执行结果（避免再一次执行）
            || !\is_object($this->values[$id]) // values不为对象、闭包（有可能具体的值）
            || isset($this->protected[$this->values[$id]]) // 受保护的服务（values为服务，不为执行结果）
            || !\method_exists($this->values[$id], '__invoke') // 是对象，但是没有__invoke方法(闭包自带该__invoke方法)
        ) {
            return $this->values[$id];
        }
        // 如果服务在工厂方法中，则直接调用（注册的是闭包）
        if (isset($this->factories[$this->values[$id]])) {
            return $this->values[$id]($this);
        }
        // 把values中的服务转存到raw中，并且冻结（防止被更改）,此时values是服务的执行结果
        $raw = $this->values[$id];
        // raw为闭包（或者带有__invoke方法的对象）
        $val = $this->values[$id] = $raw($this);
        $this->raw[$id] = $raw;
        $this->frozen[$id] = true;
        return $val;
    }

    /**
     * Checks if a parameter or an object is set.
     *
     * @param string $id The unique identifier for the parameter or object
     *
     * @return bool
     */
    public function offsetExists($id)
    {
        return isset($this->keys[$id]);
    }

    /**
     * Unsets a parameter or an object.
     *
     * @param string $id The unique identifier for the parameter or object
     */
    public function offsetUnset($id)
    {
        if (isset($this->keys[$id])) {
            // 为对象或者闭包（factories和protected存储的是对象或者闭包）
            if (\is_object($this->values[$id])) {
                unset($this->factories[$this->values[$id]], $this->protected[$this->values[$id]]);
            }
            unset($this->values[$id], $this->frozen[$id], $this->raw[$id], $this->keys[$id]);
        }
    }

    /**
     * Marks a callable as being a factory service.
     *
     * @param callable $callable A service definition to be used as a factory
     *
     * @return callable The passed callable
     *
     * @throws ExpectedInvokableException Service definition has to be a closure or an invokable object
     */
    public function factory($callable)
    {
        if (!\method_exists($callable, '__invoke')) {
            throw new ExpectedInvokableException('Service definition is not a Closure or invokable object.');
        }
        $this->factories->attach($callable);
        return $callable;
    }

    /**
     * Protects a callable from being interpreted as a service.
     *
     * This is useful when you want to store a callable as a parameter.
     *
     * @param callable $callable A callable to protect from being evaluated
     *
     * @return callable The passed callable
     *
     * @throws ExpectedInvokableException Service definition has to be a closure or an invokable object
     */
    public function protect($callable)
    {
        if (!\method_exists($callable, '__invoke')) {
            throw new ExpectedInvokableException('Callable is not a Closure or invokable object.');
        }
        $this->protected->attach($callable);

        return $callable;
    }

    /**
     * Gets a parameter or the closure defining an object.
     *
     * @param string $id The unique identifier for the parameter or object
     *
     * @return mixed The value of the parameter or the closure defining an object
     *
     * @throws UnknownIdentifierException If the identifier is not defined
     */
    public function raw($id)
    {
        if (!isset($this->keys[$id])) {
            throw new UnknownIdentifierException($id);
        }
        // 存在则说明已经获取过一次，直接返回服务，此时values为服务的执行结果
        if (isset($this->raw[$id])) {
            return $this->raw[$id];
        }
        // 不存在raw中，说明offsetGet方法没有调用过
        return $this->values[$id];
    }

    /**
     * Extends an object definition.
     *
     * Useful when you want to extend an existing object definition,
     * without necessarily loading that object.
     *
     * @param string   $id       The unique identifier for the object
     * @param callable $callable A service definition to extend the original
     *
     * @return callable The wrapped callable
     *
     * @throws UnknownIdentifierException        If the identifier is not defined
     * @throws FrozenServiceException            If the service is frozen
     * @throws InvalidServiceIdentifierException If the identifier belongs to a parameter
     * @throws ExpectedInvokableException        If the extension callable is not a closure or an invokable object
     */
    public function extend($id, $callable)
    {
        if (!isset($this->keys[$id])) {
            throw new UnknownIdentifierException($id);
        }
        // 冻结的对象不能扩展
        if (isset($this->frozen[$id])) {
            throw new FrozenServiceException($id);
        }
        // 不是对象或者闭包不能扩展（对象和闭包都可以通过is_object验证，但是对象不自带__invoke魔术方法）
        if (!\is_object($this->values[$id]) || !\method_exists($this->values[$id], '__invoke')) {
            throw new InvalidServiceIdentifierException($id);
        }
        // 受保护的对象不能扩展
        if (isset($this->protected[$this->values[$id]])) {
            @\trigger_error(\sprintf('How Pimple behaves when extending protected closures will be fixed in Pimple 4. Are you sure "%s" should be protected?', $id), \E_USER_DEPRECATED);
        }
        // $callable不是对象或者闭包不能扩展
        if (!\is_object($callable) || !\method_exists($callable, '__invoke')) {
            throw new ExpectedInvokableException('Extension service definition is not a Closure or invokable object.');
        }
        $factory = $this->values[$id];
        // 灵魂,可以看出扩展时候callable有两个参数，第一个是原对象，第二个是容器对象，有点像装饰器模式
        $extended = function ($c) use ($callable, $factory) {
            return $callable($factory($c), $c);
        };
        // 将factory拿掉，直接使用extended
        if (isset($this->factories[$factory])) {
            $this->factories->detach($factory);
            $this->factories->attach($extended);
        }
        return $this[$id] = $extended;
    }

    /**
     * Returns all defined value names.
     *
     * @return array An array of value names
     */
    public function keys()
    {
        return \array_keys($this->values);
    }

    /**
     * Registers a service provider.
     *
     * @param ServiceProviderInterface $provider A ServiceProviderInterface instance
     * @param array                    $values   An array of values that customizes the provider
     *
     * @return static
     */
    public function register(ServiceProviderInterface $provider, array $values = array())
    {
        $provider->register($this);
        foreach ($values as $key => $value) {
            $this[$key] = $value;
        }
        return $this;
    }
}
```

## 示例
1. 参考2给出了具体的一些使用姿势，参考4给出了pimple的一些优点
2. easywechat[https://github.com/overtrue/wechat](https://github.com/overtrue/wechat)

## 补充

省去了许多手动处理实例化和彼此间的依赖，比如参考2中的例子，可以相当于一个小型的自动加工线，想要什么可以丢去生产线得到。

```php
// define some services
$container['session_storage'] = function ($c) {
    return new SessionStorage('SESSION_ID');
};
$container['session'] = function ($c) {
    return new Session($c['session_storage']);
};
// get the session object
$session = $container['session'];

// 在以前，我们会采用下面的方法进行实例化
// $storage = new SessionStorage('SESSION_ID');
// $session = new Session($storage);
```

## 参考
1. [https://www.insp.top/article/realization-of-pipeline-component-for-laravel#page-break-anchor](https://www.insp.top/article/realization-of-pipeline-component-for-laravel#page-break-anchor)
2. [https://pimple.symfony.com/](https://pimple.symfony.com/)
3. [https://www.php.net/manual/zh/language.oop5.magic.php#object.invoke](https://www.php.net/manual/zh/language.oop5.magic.php#object.invoke)
4. [https://jtreminio.com/blog/an-introduction-to-pimple-and-service-containers/](https://jtreminio.com/blog/an-introduction-to-pimple-and-service-containers/)