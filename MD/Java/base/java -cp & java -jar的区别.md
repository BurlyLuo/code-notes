# java -cp & java -jar的区别

## java -cp

java -cp 和 -classpath 一样，是指定类运行所依赖其他类的路径，通常是类库和jar包，需要全路径到jar包，多个jar包之间连接符：

- Window上分号  `;`
- Linux下使用 `:`

**windows环境示例**：

```bash
java -cp .;d:\work\other.jar;d:\work\my.jar packname.mainclassname
```

**linux环境示例**：

```bash
java -cp .:/hone/myuser/work/other.jar:/hone/myuser/work/my.jar packname.mainclassname
```

**表达式支持通配符**:

例如：

```bash
java -cp .;d:\work\my.jar;d:\work\*.jar packname.mainclassname
java -cp .:/home/myuser/work/lib/my.jar:/home/myuser/work/dependency_jars/*.jar packname.mainclassname
```

## java -jar

例：

```bash
java -jar my.jar
```

执行该命令时，会用到目录`META-INF\MANIFEST.MF`文件，在该文件中，有一个叫`Main－Class`的参数，它说明了java -jar命令执行的类。

java -jar方式不可以指定附加依赖jar包。

**备注**：

- 打包时指定了主类，可以直接用java -jar {xxx.jar}。
- 打包时没有指定主类，可以用java -cp {xxx.jar} {主类名称（绝对路径）}。
- 要引用其他的jar包，可以用java -{[classpath|cp]} {$CLASSPATH}:{xxxx.jar} {主类名称（绝对路径）}。其中 -classpath 指定需要引入的类。

## Spring Boot项目jar包部署方式

方式一：

```bash
java -jar myapp.jar
```

方式二：

```bash
jar -xf myapp.jar
java org.springframework.boot.loader.JarLauncher
```

方式三：

```bash
jar -xf myapp.jar
java -cp BOOT-INF/classes:BOOT-INF/lib/* com.example.MyApplication
```
