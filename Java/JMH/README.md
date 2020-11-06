# JMH 
# Java Microbenchmark Harness 提供多线程测试
官方样例：
http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/

1.添加依赖
```xml
       <dependency>
            <groupId>org.openjdk.jmh</groupId>
            <artifactId>jmh-core</artifactId>
            <version>1.26</version>
        </dependency>
        <dependency>
            <groupId>org.openjdk.jmh</groupId>
            <artifactId>jmh-generator-annprocess</artifactId>
            <version>1.26</version>
        </dependency>

```

2.测试用例
```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.results.format.ResultFormatType;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

/**
 * BenchmarkMode模式：AverageTime - 调用的平均时间
 *                   SampleTime - 随机取样，最后输出取样结果的分布
 *                   SingleShotTime - 只会执行一次，通常用来测试冷启动时候的性能。
 *                   All - 所有的benchmark modes。
 *
 *  Threads 每个进程中的测试线程，可用于类或者方法上。
 *  Warmup 预热 :
 *           iterations：预热的次数
 *          time：每次预热的时间
 *           timeUnit：时间的单位，默认秒
 *           batchSize：批处理大小，每次操作调用几次方法
 *       为什么需要预热？
 *       因为 JVM 的 JIT 机制的存在，如果某个函数被调用多次之后，JVM 会尝试将其编译为机器码，从而提高执行速度，所以为了让 benchmark 的结果更加接近真实情况就需要进行预热
 *  Fork  需要运行的试验(迭代集合)数量。每个试验运行在单独的JVM进程中。也可以指定(额外的)JVM参数
 *  Measurement 提供真正的测试阶段参数。指定迭代的次数，每次迭代的运行时间和每次迭代测试调用的数量(通常使用@BenchmarkMode(Mode.SingleShotTime)测试一组操作的开销——而不使用循环)
 *  Scope有三种：
 *        Scope.Thread：默认的State，每个测试线程分配一个实例；
 *        Scope.Benchmark：所有测试线程共享一个实例，用于测试有状态实例在多线程共享下的性能；
 *        Scope.Group：每个线程组共享一个实例；
 *  OutputTimeUnit 为统计结果的时间单位，可用于类或者方法注解
 *  Param  指定某项参数的多种情况，特别适合用来测试一个函数在不同的参数输入的情况下的性能，只能作用在字段上，使用该注解必须定义 @State 注解。
 *
 */
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 3, time = 1)
@Measurement(iterations = 5, time = 5)
@Threads(4)
@Fork(1)
@State(value = Scope.Benchmark)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMH {

    @Param(value = {"10", "50", "100"})
    private int length;

    @Benchmark
    public void testStringAdd(Blackhole blackhole) {
        String a = "";
        for (int i = 0; i < length; i++) {
            a += i;
        }
        blackhole.consume(a);
    }

    @Benchmark
    public void testStringBuilderAdd(Blackhole blackhole) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < length; i++) {
            sb.append(i);
        }
        blackhole.consume(sb.toString());
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMH.class.getSimpleName())
                .result("result.json")
                .resultFormat(ResultFormatType.JSON).build();
        new Runner(opt).run();
    }

}

```
