# Spring Boot项目长时间运行无调用导致tmp目录被清空

## 问题

项目部署在CentOS7服务器上，长时间运行期间没有被调用，之后调用该项目服务接口时报错：

```log
java.io.IOException: The temporary upload location [/tmp/tomcat.xxx.8080/work/Tomcat/localhost/ROOT] is not valid
```

## 原因

CentOS 7 会清理 10 天前未更新的 `/tmp` 目录的文件。 springboot 框架启动后，创建的 `/tmp/tomcat.*` 目录正好在清理策略内，所以会被自动清理。

## 解决办法

方法一：Spring Boot项目配置中`server.tomcat.basedir`指向其他目录

```properties
server.tomcat.basedir=/xxx/tmp
```

注意：推荐在启动脚本上，每次启动 jar 服务前，自动删除缓存文件。

方法二：/tmp/systemd-private-%b-* 这个目录是不会被自动清理的，所以把临时目录名配置成这样，可以绕过此问题

```properties
server.tomcat.basedir=/tmp/systemd-private-8080-tomcat.service-springboot
```

方法三：启动时增加参数，指定临时目录为其他目录

```bash
-Djava.io.tmpdir=/xxx/tmp
```

方法四：修改系统设置，不清理 /tmp/tomcat* 目录

```bash
echo "x /tmp/tomcat*" > /usr/lib/tmpfiles.d/tomcat.conf
```

## 参考

- [spring boot /tmp 目录被清空](https://www.jianshu.com/p/88b04815f043)
