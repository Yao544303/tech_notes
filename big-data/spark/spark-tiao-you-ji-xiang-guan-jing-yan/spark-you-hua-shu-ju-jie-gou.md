# Spark优化数据结构

## Spark 优化数据结构

### 数据结构耗费内存情况

1. 每个Java对象，都有一个对象头，会占用16个字节，主要是包括了一些对象的元信息，比如指向它的类的指针。
2. Java的String对象，会比它内部的原始数据，多出40个字节。因为它内部使用的char数组来保存内部的字符序列。并且还得保存诸如数组长度之类的信息。
3. Java中的结合类型，比如HashMap和LinkedList,内部使用的是链表数据结构,所以对链表中的每一个数据,都使用了Entry对象来包装,Entry对象不光有对象头,还有指向下一个Entry的指针,通常占用8个字节。
4. 元素类型为原始数据类型(比如int)的集合,内部通常会使用原始数据类型的包装类型,比如Integer,来存储元素。

### 适用场景

#### 优先使用数组以及字符串,能不用集合类的,就不用集合类(优先使用array,而不是ArrayList、LinkedList、HashMap等集合)

示例： 对于HashMap、List这种数据,统一用String拼接成特殊格式的字符串,比如Map\<Integer,Person> person = new HashMap\<Integer,Person>() 可以优化为,特殊的字符串格式: id:name.address|id:name,address... 原因： array既比List少了额外信息的存储开销,还能使用原始数据类型(int)来存储数据,比List中用Integer这种包装类型存储数据,要节省内存的多

#### 应避免使用多层嵌套的对象结构

比如：

```
public class Teacher{private list<Student> students = new ArrayList<Student>()}
```

这就是一个非常不好的例子,因为Teacher类的内部又嵌套了大量的小student对象

#### 对于有些能够避免的场景，尽量使用int替代String

因为String虽然比ArrayList、HashMap等数据结构高效、占用内存少、但是String还是有额外信息的消耗,比如之前用String表示id,那么现在完全可以用数字类型的int,来进行替代(注:id不要用常用的uuid,因为无法转成int,就用自增类型的int类型的id即可)

## Spark高效数据结构

### BitSet

```
org.apache.spark.util.collection.BitSet是一个简单，大小不可变的bit set实现。
```

BitSet利用一个Long型words数组来存储数值 优势:

* 运算效率高。 get和set过程都采用移位运算，提高运算效率。
* 占有的存储空间少。 一个Long型可以存储64个数值。那么如果N = 10,000,000，需要N/8 = 10,000,000/8 Byte 约等 1.25M

### OpenHashSet

### OpenHashMap

## ref

* [Spark如何优化数据结构](https://blog.csdn.net/qq\_35478489/article/details/78816119)
* [Spark高效数据结构](https://www.jianshu.com/p/de8bb509b6f5)
* [Added a number of very fast, memory-efficient data structures](https://gite.lirmm.fr/yagoubi/spark/commit/3e7df8f6c6edec9e71c6e416de86aa1a2cefc176)
