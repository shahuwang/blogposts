## Java单例模式最佳实现方式

前段时间，偶然在Stackoverflow上看到了一个问题，问的是如何实现单例模式是最好的。得票率最高的答案是使用enum方式。有兴趣可以看原文[What is an efficient way to implement a singleton pattern in Java?](http://stackoverflow.com/questions/70689/what-is-an-efficient-way-to-implement-a-singleton-pattern-in-java)  

原文里面涉及到了很多知识点我还不明白，所以这里暂时不会涉及那么深的方面。主要是记录一下，以便自己以后使用。我自己写的一个简单的单例实现如下：  
    
    public enum Foo {
          INSTANCE;
         public Foo getInstance(){
         //增加这个方法是让别人明白怎么使用，因为这种实现方式
          //还比较少见，限于java 1.5之后的版本
         return INSTANCE;
    }
     public void print(){
     System.out.println("hello,world");
     }
    }

这个enum类型，实现了两个方法，getInstance和print。其实也可以直进Foo.INSTANCE获取到单例。使用方法如下：

    public class FooUse {
       public static void main(String[] args){
       Foo f = Foo.INSTANCE;
       f.print();
       }
    }

关于单例的其他实现方式，可以看这篇[Java:单例模式的七种写法](http://www.blogjava.net/kenzhh/archive/2013/03/15/357824.html) 。文章里面介绍了各种写法的利弊，不过涉及的内容并不深入。有需要的可以去找更多的文章来看。

关于enum类型，我学习Java以来，很少用到，也很少见到，所以对它也不是很了解。不过网上永远都是有非常多的人写得好文章免费供大家学习的。关于enum的内容，可以看这篇[Java 语言中 Enum 类型的使用介绍](http://www.ibm.com/developerworks/cn/java/j-lo-enum/) ,以及这篇[java枚举类型enum的使用 ](http://blog.csdn.net/wgw335363240/article/details/6359614)。后面这篇比前面那篇多讲述了枚举构造函数的作用以及使用：

    public enum Light {

       // 利用构造函数传参

       RED (1), GREEN (3), YELLOW (2);

 

       // 定义私有变量

       private int nCode ;

 

       // 构造函数，枚举类型只能为私有

       private Light( int _nCode) {

           this . nCode = _nCode;

       }

 

       @Override

       public String toString() {

           return String.valueOf ( this . nCode );

       }

    } 

这里的RED(1)就是调用了构造函数。这个类主要的功能，是把枚举值输出。譬如RED的值到底是什么呢？通过构造函数RED（1），我们知道RED的值是1 。当然也可以是RED（“红灯”）这样的实现方式。Java貌似会默认提供1的值。

至于为什么说用枚举实现的singleton是最好的，我现在的知识能力还无法解答。目前仅知道，enum里面的枚举数据都是线程安全的，而enum实现的单例又是最简单的，这是Effective Java的作者推荐的方式