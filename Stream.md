# Java Stream

Stream(流)是一个来自数据源的元素队列并支持聚合操作

1. 元素是特定类型的对象，形成一个队列。Java中的Stream并不会存储元素，而是按需计算
2. 数据源 流的来源。 可以是集合，数组，I/O channel，产生器generator等
3. 聚合操作 类似SQL语句一样的操作，比如filter，map，reduce，find，match，sorted等



和以前的Collection操作不同，Stream操作还有两个基础的特征：

1. Pipelining：中间操作都会返回流对象本身。这样多个操作可以串联成一个管道，如同流式风格。这样做可以对操作进行优化，比如延迟执行和短路
2. 内部迭代：以前对集合遍历都是通过Iterator或者For-Each的方式，显式的在集合外部进行迭代，这叫做外部迭代。Stream提供了内部迭代的方式，通过访问者模式实现



例子：

```java
import java.util.Arrays;
import java.util.IntSummaryStatistics;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;
public class StreamDemo {
    public static void main(String[] args) {
    	//生成数组
        List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
        List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);

        //串行流创建
        List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());

        //Stream 提供forEach来迭代流中的每个数据
        Random random = new Random();
        random.ints().limit(10).forEach(System.out::println);

        //map 方法用于映射每个元素到对应的结果
        List<Integer> squaresList = numbers.stream().map(i -> i*i).distinct().collect(Collectors.toList());
        System.out.println(squaresList);

        //filter 方法用于通过设置的条件过滤出元素
        long count = strings.stream().filter(string -> string.isEmpty()).count();
        System.out.println(count);

        //limit 方法用于获取指定数量的流(略)
        //sorted 方法用于对流进行排序(略)


        //并行（parallel）程序
        long count1 = strings.parallelStream().filter(string -> string.isEmpty()).count();
        System.out.println(count1);

        //Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素
        List<String> filtered1 = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
        System.out.println("筛选列表: " + filtered1);
        String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
        System.out.println("合并字符串: " + mergedString);

        //统计
        IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
        System.out.println("列表中最大的数 : " + stats.getMax());
        System.out.println("列表中最小的数 : " + stats.getMin());
        System.out.println("所有数之和 : " + stats.getSum());
        System.out.println("平均数 : " + stats.getAverage());
    }
}
```