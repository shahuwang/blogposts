## Maven使用本地jar包并打包进war包里面的方法

很显然，这种方法是很不可取的，因为Maven是用来团队合作，以及发布开源代码的。而使用本地jar包，则使得Maven丧失了这部分的优点。不过，我遇到的问题是，我想学习Maven，然后我以前的项目，公司的项目都不是用Maven的。然后我想引用其中的一些jar包，特别是某些项目build之后在dist文件夹下的包。所以，才想出了下面的法子。

我先在我的Maven项目下建立一个lib文件夹，把我要用到的jar包都放到里面去。然后在pom.xml里添加如下的内容：

     <dependency>
    <groupId>com.weiresearch</groupId>
    <artifactId>webharvest</artifactId>
    <version>1.0.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/webharvest.jar</systemPath>
    </dependency>

这里的groupId，artifactId，version都可以随便写。然后scope要写成system，systemPath就写为要引用的jar包路径。这里不知道能否批量导入，没测试过，不过貌似不可以。

但是，还有个麻烦问题，我把我的项目打包成war包的时候，它居然只有我写的代码，而不包括引用的jar包。这就很麻烦了，打包的war包就不能直接使用了。找来好久，终于找到原因了。首先是其他正常使用maven的包，如下：
    
     <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
      </dependency>
这里引用的是servlet包，最重要的是scope这个字段，provided的意思，就是说这个jar包，在这个项目可能的运行环境里，都会提供了的，所以就不用打包到war包里面了。因此，要使得war包里面包含servlet这个包，只要把scope这句给删除掉就可以了。

而上面利用system这个scope引入的webharvest包，同样也没有引入进去。看了下网上的评论，才说最好不要用system这个scope。大概是说这个system指的是JDK里面会包含这个jar包吧。这个解决方法就麻烦人了，最后是看到了这个问答才解决了的：[Maven 2 assembly with dependencies: jar under scope “system” not included](http://stackoverflow.com/questions/2065928/maven-2-assembly-with-dependencies-jar-under-scope-system-not-included) ，解决方法如下：

     <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <webResources>
                        <resource>
                            <directory>${project.basedir}/lib</directory>
                            <targetPath>WEB-INF/lib</targetPath>
                            <filtering>true</filtering>
                            <includes>
                                <include>**/*.jar</include>
                            </includes>
                        </resource>
                    </webResources>
                </configuration>
     </plugin>

directory字段指向的是包含你所有要用jar包的目录
 
targetPath则是编译后要把这些jar包复制到的位置

下面的filtering就是只导入所有以jar为后缀的文件

通过上面的设置，就能把你要的jar包导入进去了