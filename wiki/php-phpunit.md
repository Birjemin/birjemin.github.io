# Phpunit

## 简介

## 安装
`一般composer安装，vendor/bin/phpunit` 

## 编写 PHPUnit 测试

### 测试的依赖关系
用 @depends 标注来表达依赖关系（`DependTest.php`）

### 数据供给器
用 @dataProvider 标注来表达数据供给器（`AdditionProviderTest.php`）
同一个测试中组合使用 @depends 和 @dataProvider（`DependencyAndDataProviderComboTest.php`）

### 对异常进行测试
用 @expectException 标注来测试被测代码中是否抛出了异常（`ExceptionTest.php`）

### 对输出进行测试
用 expectOutputString() 方法来设定所预期的输出

| 方法 | 含义 |
| :---: | :---: |
| void expectOutputRegex(string $regularExpression) | 设置输出预期为输出应当匹配正则表达式 $regularExpression。|
| void expectOutputString(string $expectedString) | 设置输出预期为输出应当与 $expectedString 字符串相等。|
| bool setOutputCallback(callable $callback) | 设置回调函数，用来做诸如将实际输出规范化之类的动作。|

## 命令行测试执行器
手册只是列出了phpunit --help，自己看吧

## 基境(fixture)
在编写测试时，最费时的部分之一是编写代码来将整个场景设置成某个已知的状态，并在测试结束后将其复原到初始状态。这个已知的状态称为测试的基境(fixture)。

### 一般操作
PHPUnit 支持共享建立基境的代码。在运行某个测试方法前，会调用一个名叫 setUp() 的 模板方法。setUp() 是创建测试所用对象的地方。当测试方法运行结束后，不管是成功还是失败，都会调用另外一个名叫 tearDown() 的模板方法。另外，setUpBeforeClass() 与 tearDownAfterClass() 模板方法将分别在测试用例类的第一个测试运行之前和测试用例类的最后一个测试运行之后调用。（`TemplateMethodsTest.php`）

用 setUp() 建立栈的基境（`StackTest.php`）

### 基境共享
在测试之间共享基境会降低测试的价值。潜在的设计问题是对象之间并 非松散耦合。如果解决掉潜在的设计问题并使用桩件(stub)(参见第 9 章 测试替身)来编写测试，就能达成更好的结果，而不是在测试之间产生运行时依赖并错过改进设计的机会。（`DatabaseTest.php`）

## 组织测试
PHPUnit 的目标之一是测试应当可组合:我们希望能将任意数量的测试以任意组合方式运行。

## 有风险的测试
## 未完成的测试与跳过的测试
### 未完成的测试
比如代码没写完啊，功能测试没写完啊。(`SampleTest.php`)

| 方法 | 含义 |
| :---: | :---: |
| void markTestIncomplete() | 将当前测试标记为未完成。|
| void markTestIncomplete(string $message) | 将当前测试标记为未完成，并用 $message 作为说明信息。 |

### 跳过测试
(`Sample1Test.php`)

| 方法 | 含义 |
| :---: | :---: |
| void markTestSkipped() | 将当前测试标记为已跳过。 |
| void markTestSkipped(string $message) | 将当前测试标记为已跳过，并用 $message 作为说明信息。 |

`@requires` 来跳过测试

## 数据库测试
1. 建立基境(fixture)
2. 执行被测系统
3. 验证结果
4. 拆除基境(fixture)

(MyApp_Tests_DatabaseTestCase1.php只做了几个常用的方式，其他的没弄)
* Flat XML DataSet (平直 XML 数据集)
* XML DataSet (XML 数据集)
* MySQL XML DataSet (MySQL XML 数据集)
* YAML DataSet (YAML 数据集)
* CSV DataSet (CSV 数据集)
* Array DataSe (数组数据集)
* Query (SQL) DataSet (查询(SQL)数据集)
* Database (DB) Dataset (数据库数据集)

## 测试替身
Stubs (桩件)
将对象替换为(可选地)返回配置好的返回值的测试替身的实践方法称为上桩(stubbing)。可以用桩件(stub)来“替换掉被测系统所依赖的实际组件，这样测试就有了对被测系统的间接输入的控制点。这使得测试能强制安排被测系统的执行路径，否则被测系统可能无法执行”。




## 备注

## 参考
