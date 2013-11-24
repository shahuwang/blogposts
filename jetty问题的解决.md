今天运行jetty部署应用的时候，遇到了这样的问题：

org.apache.jasper.JasperException: PWC6345: There is an error in invoking javac.  A full JDK (not just JRE) is required

一时不知道什么原因，就把jdk重装了，还是不能解决。最后在官方的页面上找到了解决方法：http://wiki.eclipse.org/Jetty/Howto/Configure_JSP

把   -Dorg.apache.jasper.compiler.disablejsr199=true  这句放到start.ini 这个文件里面，然后启动就可以解决问题了
