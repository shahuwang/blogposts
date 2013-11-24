##  Java中用匿名内部类实现实例化抽象类

今天看scriptella的源代码，看到一段很诡异的代码，如下：

     private static final Comparator<String> ASCII_CASE_INSENSITIVE_ORDER = new Comparator<String>() {
        @Override
        public int compare(String s1, String s2) {
            int n1 = s1.length(), n2 = s2.length();
            int n = n1 < n2 ? n1 : n2;
            for (int i = 0; i < n; i++) {
                char c1 = s1.charAt(i);
                char c2 = s2.charAt(i);
                if (c1 != c2) {
                    if (c1 >= 'A' && c1 <= 'Z') { //Fast lower case
                        c1 = (char) (c1 | 0x20);
                    }
                    if (c2 >= 'A' && c2 <= 'Z') {
                        c2 = (char) (c2 | 0x20);
                    }
                    if (c1 != c2) {
                        return c1 - c2;
                    }
                }
            }
            return n1 - n2;
        }
    };

这段代码，在进行new实例化对象的时候，居然还在里面实现一些方法，很奇怪。一下子没看明白。然后看到了这篇[《 Java中用匿名内部类实现实例化抽象类》](http://cnn237111.blog.51cto.com/2359144/1131550) ，总算是搞明白了。

首先，Comparator是一个抽象类。有一点是可以明确的，抽象类是不能实例化的。而为了实例化，必须对抽象类进行实现。上面的代码利用的是匿名内部类。也就是new之后还有{} ，这里就说明它实现了一个匿名的内部类，然后这个内部类是继承于Comparator的。通过这种方式，就实现了临时性的抽象类实现，然后就可以实例化了。这样的好处就是减少了很多的代码，至少减少了一个类。