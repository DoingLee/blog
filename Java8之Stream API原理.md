我们知道，stream是把中间操作都记录下来，最后在结束操作时，才真正执行所有操作逻辑。

中间操作和结束操作在上一篇（）已经描述过

（图）


所以stream底层是如何实现的呢？

我们在使用 Stream API时，基本上所有中间操作，都会传入一个Lambda表达式，也就是回调函数。

因此，一个完整的操作应该是由<数据来源，操作框架，回调函数>构成的三个元素组成：

- 数据来源：我们的集合容器
- 操作框架：Stream API提供的各种接口，比如map、filter等
- 回调函数：我们传入的Lambda表达式函数，供操作框架回调使用

以下面代码为例：


```
Stream.of("a1", "b2", "c3", "dd4")
        .filter(s -> { // 无状态
            System.out.println("filter : " + s);
            return s.length() < 3;
        })
        .map(s -> { // 无状态
            System.out.println("map1 : " + s);
            return Integer.parseInt(s.substring(1));
        })
        .sorted((i1, i2) -> { // 有状态
            System.out.println("sort : " + i1 + " and " + i2);
            return i1 - i2;
        })
        .map(i -> { // 无状态
            System.out.println("map2 : " + i);
            return Integer.toString(i);
        })
        .forEach(s -> { // 结束
                System.out.println("forEach : " + s);
        });
```

Stream API框架里，使用Stage记录每一个操作：

- Head用于表示第一个Stage，即在调用诸如Collection.stream()方法产生的Stage，很显然这个Stage里不包含任何操作
- StatelessOp和StatefulOp分别表示无状态和有状态的Stage，对应于无状态和有状态的中间操作

多个Stage以双向链表的形式组织在一起，构成整个流水线：每个Stage都记录了前一个Stage和本次的操作以及回调函数，依靠这种结构，就能记录下对数据源的所有操作。

（图）

由于操作记录链表中前面的Stage并不知道后面Stage到底执行了哪种操作，以及回调函数是哪种形式。也就是说，只有当前Stage本身才知道该如何执行自己包含的动作。这就需要有某种协议来协调相邻Stage之间的调用关系。

JDK中使用Sink接口实现这种协议：

（图）

有了Sink接口协议，每个Stage都会将自己的操作封装到一个Sink里，前一个Stage只需调用后一个Stage的accept()方法即可，并不需要知道其内部是如何处理的。

- 对于有状态的操作，Sink的begin()和end()方法是必须实现的。比如Stream.sorted()是一个有状态的中间操作，其对应的Sink.begin()方法可能创建一个乘放结果的容器，而accept()方法负责将元素添加到该容器，最后end()负责对容器进行排序。
- 对于短路操作，Sink.cancellationRequested()是必须实现的，比如Stream.findFirst()是短路操作，只要找到一个元素，cancellationRequested()就应该返回true，以便调用者尽快结束查找。

有了Sink对操作的包装，Stage之间的调用问题就解决了，执行时只需要从流水线的head开始对数据源依次调用每个Stage对应的Sink.{begin(), accept(), cancellationRequested(), end()}方法就可以了。

Sink的四个接口方法常常相互协作，共同完成计算任务。实际上Stream API操作框架实现的的本质，就是如何重载Sink的这四个接口方法，实现不同的操作控制。

（图）


最后，在结束操作触发时，便可顺着操作记录链表，通过Sink把所有stage的操作触发：

（图）

把前面的代码运行输出结果如下：

```
filter : a1
map1 : a1
filter : b2
map1 : b2
filter : c3
map1 : c3
filter : dd4
sort : 2 and 1
sort : 3 and 2
map2 : 1
forEach : 1
map2 : 2
forEach : 2
map2 : 3
forEach : 3
```


参考：

- https://www.cnblogs.com/CarpenterLee/p/6637118.html
- https://www.cnblogs.com/Dorae/p/7779246.html?
- 文中图来源网络，作者

（末尾图）


