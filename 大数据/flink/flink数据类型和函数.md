## Rich函数类

```
public abstract class RichFlatMapFunction<IN, OUT> extends AbstractRichFunction implements FlatMapFunction<IN, OUT>
```

它既实现了`FlatMapFunction`接口类，又继承了`AbstractRichFunction`。其中`AbstractRichFunction`是一个抽象类，有一个成员变量`RuntimeContext`，有`open`、`close`和`getRuntimeContext`等方法。



我们尝试继承并实现RichFlatMapFunction`，并使用一个累加器。累加器的概念：在单机环境下，我们可以用一个for循环做累加统计，但是在分布式计算环境下，计算是分布在多台节点上的，每个节点处理一部分数据，因此单纯循环无法满足计算。累加器是大数据框架帮我们实现的一种机制，允许我们在多节点上进行累加统计。

```java
// 实现RichFlatMapFunction类
// 添加了累加器 Accumulator
public static class WordSplitRichFlatMap extends RichFlatMapFunction<String, String> {

    private int limit;

    // 创建一个累加器
    private IntCounter numOfLines = new IntCounter(0);

    public WordSplitRichFlatMap(Integer limit) {
        this.limit = limit;
    }

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        // 在RuntimeContext中注册累加器
        getRuntimeContext().addAccumulator("num-of-lines", this.numOfLines);
    }

    @Override
    public void flatMap(String input, Collector<String> collector) throws Exception {

        // 运行过程中调用累加器
        this.numOfLines.add(1);

        if(input.length() > limit) {
            for (String word: input.split(" "))
            collector.collect(word);
        }
    }
}
```

在主逻辑中获取作业执行的结果，得到累加器中的值。

```java

// 获取作业执行结果
JobExecutionResult jobExecutionResult = senv.execute("basic flatMap transformation");
// 执行结束后 获取累加器的结果
Integer lines = jobExecutionResult.getAccumulatorResult("num-of-lines");
System.out.println("num of lines: " + lines);
```

RichFunction函数类最具特色的功能是有状态计算。

