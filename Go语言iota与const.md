## Go语言iota与const

这几天看Tiedot这个NoSQL的源代码，看得我真是头疼，作者估计是个 C 语言高手，写的 Go 语言作品也充满了 C 语言的味道，用了很多位运算的东西。另外，一处关于 iota 和 const 的用法，也让我甚是迷惑。然后就用 Google 搜索了一下，发现 Go 语言里面，iota 和 const 的组合还是有许多知识点要学习的。

你可以看看这篇 [《go中iota的用法》](http://my.oschina.net/0757/blog/66765) , 以及 [《 GO语言 iota  》](http://blog.163.com/huv520@126/blog/static/2776523920100313211221/) 你会发现，const 和 iota 的组合，还有许多奇怪的要注意的用法。

const 在 Go 语言里，用来在定义常量的时候使用。而 iota，则是一个枚举数，但是它只能在 const 里面使用，可以说，iota 是 const 结构里面，定义常量行数的索引器。如下代码：

     const (
    	A = 1
    	C = 2
    	D = iota
    	
    )
你觉得 D 应该等于多少？记住一点，iota 是属于当前 const 里面的行数索引器，所以，此处 iota 在第三行，所以 iota == 2 。

再来一段如下：

    const (
    	A = 1
    	C = 2
    	D = iota
    	B
    )

这里的 B 应该等于多少？ 这里其实是使用了 Go 语言的一些简化的写法（不过我真觉得没有多少意义的）。 此处的 B 也是和上面的A，C，D一样是个表达式，有右值的，其右值就是它上面一行的右值。也就是 D=iota 右值。那么 B == 3？ 显然不对，此处 B 等于 D=iota 的右值，所以 B=iota，此处是第四行，所以为 4 。

那么下面这段代码里面，E 的值是多少呢？

    const (
    	A = 1
    	C = 2
    	D = iota
    	B
    	E
    )

答案是 4. 因为 E 的右值是 B 的右值，也就是 iota。此处是第五行，索引是 4.

    const (
      a = 1 << iota  // a == 1 (iota has been reset)1*2^0
      b = 1 << iota  // b == 2  1*2^1
      c 
    )

上面这段里面，c 的值是多少呢？ c 的右值就是 b 的右值，也就是 1 << iota ,此时 iota 的值是 2，所以此时 c 的值是 4 。

那么，如下里的两个变量，值如何？

    const x = iota  
    const y = iota 
其实，x==0， y==0 。这是为什么呢？因为 iota 里，在每一个 const 里面，都是一个新的。
综上所述，iota 的用法应该可以明确了：

1. 只能在 const 里面使用
2. 是 const 里面的行数索引器
3. 每个 const 里面，iota 都从 0 开始

看了那么多代码，你是否有注意到一些事情呢？

首先，在 const 里面，赋值是用 = 号的，而不是正常的 := .。

找到的这篇 [http://www.cnblogs.com/howDo/archive/2013/04/15/GoLang-Constant.html](http://www.cnblogs.com/howDo/archive/2013/04/15/GoLang-Constant.html) 是比较好的一篇文章。

const 里面声明常量，有如下要求：

+ 其类型必须是：数值、字符串、布尔值
+ 表达式必须是在编译期可计算的
+ 声明常量的同时必须进行初始化，其值不可再次修改

初始化用的是 = 号，不知道为什么。

还有，const 有无类型常量的东西，完整的关于  const 的说明，参考 这篇 [《常量》](http://www.ituring.com.cn/article/40964) 感觉相当不错。

