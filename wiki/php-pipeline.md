# Pipeline管道模型
## 名词简介

```
It’s an architectural pattern which encapsulates sequential processes. When used, it allows you to mix and match operation, and pipelines, to create new execution chains. 
```
嗯。。没看懂，不过从其他相关资料有了一个大致的理解，该模型形同一节一节通道，通过阀门连接，比如有通道（A，B，C），数据从A->B->C，如果通过通道A的验证，则推入B通道..以此类推，貌似一节节管道连接在一起一样。

## 官方示例

```
use League\Pipeline\Pipeline;

class TimesTwoStage
{
    public function __invoke($payload)
    {
        return $payload * 2;
    }
}

class AddOneStage
{
    public function __invoke($payload)
    {
        return $payload + 1;
    }
}

$pipeline = (new Pipeline)
    ->pipe(new TimesTwoStage)
    ->pipe(new AddOneStage);

// Returns 21
$pipeline->process(10);
```

先初始化一个管道d，然后调用process方法推入初始值，经过通道`TimesTwoStage`，`AddOneStage`，最后得出结果

## 源码阅读

* Pipeline.php
```
<?php
declare(strict_types=1);

namespace League\Pipeline;

class Pipeline implements PipelineInterface
{
    /**
     * @var callable[]
     */
    private $stages = [];

    /**
     * @var ProcessorInterface
     */
    private $processor;
    // 指定processor，提供了InterruptibleProcessor和FingersCrossedProcessor两种processor，实例化时初始化stages（可不初始化）
    public function __construct(ProcessorInterface $processor = null, callable ...$stages)
    {
        $this->processor = $processor ?? new FingersCrossedProcessor;
        $this->stages = $stages;
    }
    // 推入stage
    public function pipe(callable $stage): PipelineInterface
    {
        $pipeline = clone $this;
        $pipeline->stages[] = $stage;

        return $pipeline;
    }
    // 运行队列（实际上就是循环调用pipe推入的stage）
    public function process($payload)
    {
        return $this->processor->process($payload, ...$this->stages);
    }

    public function __invoke($payload)
    {
        return $this->process($payload);
    }
}
```

* PipelineBuilder.php
封装Pipeline方法，提供add和build方法。

* 官方示例
```
$pipeline = (new Pipeline)
    ->pipe(function () {
        throw new LogicException();
    });
    
try {
    $pipeline->process($payload);
} catch(LogicException $e) {
    // Handle the exception.
}
```

```
use League\Pipeline\PipelineBuilder;

// Prepare the builder
$pipelineBuilder = (new PipelineBuilder)
    ->add(new LogicalStage)
    ->add(new AnotherStage)
    ->add(new LastStage);

// Build the pipeline
$pipeline = $pipelineBuilder->build();
```

## 思考和补充
laravel中有一套管道的使用（中间件的使用。。。），但是具体的实现和这个有一些区别，详情可以查看参考5.
具体的使用场景有很多，比如使用该模型，可以提高代码的可读性和可拓展性，当增加一个限制条件时，只需要增加一节管道就好了，而不需要修改原有的逻辑判断。

## 参考

1. [https://pipeline.thephpleague.com/](https://pipeline.thephpleague.com/)
2. [https://medium.com/@agoalofalife/pipeline-and-php-d9bb0a6370ca](https://medium.com/@agoalofalife/pipeline-and-php-d9bb0a6370ca)
3. [https://matthewdaly.co.uk/blog/2018/10/05/understanding-the-pipeline-pattern/](https://matthewdaly.co.uk/blog/2018/10/05/understanding-the-pipeline-pattern/)
4. [https://learnku.com/articles/14151/recommend-a-php-pipeline-plug-in-leaguepipeline](https://learnku.com/articles/14151/recommend-a-php-pipeline-plug-in-leaguepipeline)
5. [https://learnku.com/laravel/t/7543/pipeline-pipeline-design-paradigm-in-laravel](https://learnku.com/laravel/t/7543/pipeline-pipeline-design-paradigm-in-laravel)