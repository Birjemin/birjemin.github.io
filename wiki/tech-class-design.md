# 类的设计原则-SOLID
![示意图](http://upload.ouliu.net/i/20180303164338xtbf5.jpeg)

## 简介
SOLID:
> * S: 单一职责原则 (SRP)
> * O: 开闭原则 (OCP)
> * L: 里氏替换原则 (LSP)
> * I: 接口隔离原则 (ISP)
> * D: 依赖反转原则 (DIP)

> * 迪米特法则

## 详情

### 单一职责原则（SRP:Single responsibility principle）
解耦和增强内聚性（高内聚，低耦合），一个类和方法的只负责一个职责

* 示例1：一个类中一个方法职责混乱。

```php
class Activity
{
  public function getActivity()
  {
    if ($this->startDate < time() && $this->endDate > time()) {
      return '活动：' . $this->name . '已经在' . date('Y-m-d H:i:s', $this->startDate) . '开始';
    } else {
      return '活动：' . $this->name . '没有开始';
    }
  }
}
```

弊端：如果再增加条件，和输出修改,会加重逻辑。
改变为 ->

```php
class Activity
{
  public function getActivity()
  {
    return $this->isStart ? $this->getStartWord() : $this->getEndWord();
  }
  public function isStart()
  {
    return $this->startDate < time() && $this->endDate > time();
  }
  public function getStartWord()
  {
    return '活动：' . $this->name . '已经在' . date('Y-m-d H:i:s', $this->startDate) . '开始';  
  }
  public function getNotStartWord()
  {
    return '活动：' . $this->name . '没有开始';  
  }
}
```

* 示例2：类的职责混乱

```php
class Activity
{
  public function __construct(Activity $activity)
  {
    $this->activity = $activity;
  }
  public function draw()
  {
    if ($this->isStart()) {
      return $this->draw();
    }
    throw Exception('没有开始');
  }
}
```

弊端：类的职责不清。抽奖和活动不应该属于同一个类
更改为->

```php
class Activity
{
  public function __construct(Activity $activity)
  {
    $this->activity = $activity;
    $this->config = '38_festival';
  }
  public function draw()
  {
    if ($this->isStart()) {
      return $this->initDrawManage->draw();
    }
    throw Exception('没有开始');
  }
  public function initDrawManage()
  {
    !isset($this->drawManage) && $this->drawManage = new DrawManage($this->config)
    return $this->drawManage
  }
}
class DrawManage
{
  public function __construct($config)
  {
    $this->config = $config
  }
  public function draw()
  {
    ...
  }
}
```

### 开闭原则(OCP:Open/Closed Principle)

对扩展开放，对修改关闭。
与其修改别人的代码（或者老代码）不如先继承，然后更改。
* 示例1：新增一个学生，应当采用继承的方式来做，而不是修改原有的代码

```php
interface UserInterface
{
  public function getUserIdentify();
}
class User implements UserInterface
{
  private $identify = '先生/女士';
  public function getUserIdentify
  {
    return $this->name . $this->identify;
  }
}
```

改为->

```php
interface UserInterface
{
  public function getUserIdentify();
}
class User implements UserInterface
{
  private $identify = '先生/女士';
  public function getUserIdentify
  {
    return $this->name . $this->identify;
  }
}
class Student extend User
{
  private $identify = '学生';  
}
```

* 示例2：调整结构。
增加新的角色和工种时

```php
interface UserInterface
{
  public function getUserIdentify();
}
class User implements UserInterface
{
  private $identify = 'young';
  public function getUserIdentify()
  {
    return $this->identify;
  }
}

class Student extend User
{
  private $identify = 'student';  
}

class Work
{
  private $identify

  public function __construct(UserInterface $identify)
  {
    $this->identify = $identify;
  }

  public function getWorking()
  {
    return $this->identify->getIdentify() == 'student'
    ? $this->study() : $this->working();
  }
  public function study()
  {
    return '学习';
  }
  public function wordking()
  {
    return '工作';
  }
}
```

弊端：修改代码~~
更改Work类的代码->

```php
interface UserInterface
{
  public function getUserIdentify();
  public function doing();
}
class User implements UserInterface
{
  private $identify = 'young';
  public function getUserIdentify()
  {
    return $this->identify;
  }
  public function doing()
  {
    return 'working';
  }
}

class Student extend User
{
  private $identify = 'student';
  public function  doing()
  {
    return 'study';
  }
}

class Work
{
  private $identify

  public function __construct(UserInterface $identify)
  {
    $this->identify = $identify;
  }

  public function getWorking()
  {
    return $this->identify->doing();
  }
}
```

### 里氏替换原则（LSP:Liskov Substitution Principle）

父类出现的地方子类就可以出现，且替换成子类也不会出现任何错误或者异常。

### 接口隔离原则 (ISP:Interface Segregation Principle)

针对接口的原则，规范如下：
* 接口尽量小（细化业务）
* 接口高内聚（减少对外交互，public方法少）
* 定制服务（应单独为一个个体服务）
* 有限设计（和第一个规范相辅相成）

示例：

```php
interface User
{
  public function working();
  public function studing();
}
class Student implements User
{
  public function working()
  {
    // 学生不工作（假设）
    return null;
  }
  ...
}
class Worker implements User
{
  public fucntion studing
  {
    // 不学习（假设）
    return null;
  }
}
```

可以修改->

```php
interface StudentInterface
{
  public function studing();
}
interface WorkerInterface
{
  public function working();
}
class Student implements StudentInterface
{
  ...
}
class Worker implements WorkerInterface
{
  ...
}
```

### 依赖反转原则 (DIP:Dependency Inversion Principle)
高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。

这个原则刚好本文的第二小节开闭原则示例1貌似就违反了，但是要看具体业务的，如果是A是父类，B是子类，具有一定的联系，无法分割就不遵循。如果A类和B类是平级关系，继承只是共用一些方法，那就有必要让两者都依赖抽象，便于该改动。

### 迪米特法则（Law of Demeter）
迪米特法则也叫做最少知识原则(Least Knowledge Principle,LKP)，即一个对象应该对其他对象有最少的了解。不关心其他对象内部如何处理。

## 综述
根据实际情况使用不同的原则，可以使得程序降低耦合和冗余代码，优化程序。根据这些规则，会有一系列的设计模式，在实际使用时没必要强行套用设计模式，需要根据业务实际划分合理即可。长城不是一天就能修好的~~~
  
补充：添加参考中的第四点，大家可以看一看。

| 职责 | 作用 |
| :---: | :---: | :---: |
| 单一职责原则 | 实现类要职责单一 |
| 里氏替换原则 | 不要破坏继承体系 |
| 依赖倒置原则 | 面向接口编程 |
| 接口隔离原则 | 在设计接口的时候要精简单一 |
| 开闭原则 | 对扩展开放，对修改关闭 |
| 迪米特法则 | 降低耦合 |

## 参考
1. [https://www.cnblogs.com/HouJiao/p/5459022.html](https://www.cnblogs.com/HouJiao/p/5459022.html)
2. [https://segmentfault.com/a/1190000011662984](https://segmentfault.com/a/1190000011662984)
3. [http://blog.csdn.net/jhq0113/article/details/44907029](http://blog.csdn.net/jhq0113/article/details/44907029)
4. [https://www.jianshu.com/p/21573a0b2ad9](https://www.jianshu.com/p/21573a0b2ad9)
