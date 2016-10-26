---
title: 使用lombok简化Java代码
date: 2016-10-09 19:53:34
tags:
	- java
	- lombok
	- 代码优化
---

### 0. 简介：
	lombok是优化Java代码的一个小工具，特别是对Java Bean中的代码。

#### [val](https://projectlombok.org/features/val.html) 
	声明一个final的变量 可以做简单类型推导可参考 

#### [@NonNull](https://projectlombok.org/features/NonNull.html) 
	修饰属性时会生成一个包含对应参数的构造函数，修饰方法参数时会校验参数不能为空，为空时可抛出空指针异常

使用lombok注解的代码

```
public NonNullExample(@NonNull Person person) {
     super("Hello");
     this.name = person.getName();
}
```

实际生成的代码

```
public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person");
    }
    this.name = person.getName();
}
```

#### [@Cleanup](https://projectlombok.org/features/Cleanup.html) 
	清理释放资源代码,会在使用资源的代码中自动添加释放资源的代码

使用lombok注解的代码

```
public class CleanupExample {
   public static void main(String[] args) throws IOException {
     @Cleanup InputStream in = new FileInputStream(args[0]);
     @Cleanup OutputStream out = new FileOutputStream(args[1]);
     byte[] b = new byte[10000];
     while (true) {
       int r = in.read(b);
       if (r == -1) break;
       out.write(b, 0, r);
     }
   }
 }
```

实际生成的代码

```
 public static void main(String[] args) throws IOException {
     InputStream in = new FileInputStream(args[0]);
     try {
       OutputStream out = new FileOutputStream(args[1]);
       try {
         byte[] b = new byte[10000];
         while (true) {
           int r = in.read(b);
           if (r == -1) break;
           out.write(b, 0, r);
         }
       } finally {
         if (out != null) {
           out.close();
         }
       }
     } finally {
       if (in != null) {
         in.close();
       }
     }
   }
```


#### [@Getter / @Setter](https://projectlombok.org/features/GetterSetter.html) 
	生成getter/setter代码,会是JavaBean中的代码减少2/3

如图所示，可以修饰属性，
![](/img/lombok/lombok_01.png)

也可以修饰类
![](/img/lombok/lombok_02.png)

#### [@ToString](https://projectlombok.org/features/ToString.html)
	 生成tostring代码 @ToString(xxx=xxx)可以加参数，具体参考官方文档

#### [@EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode.html)
	 生成equals方法和hashCode方法

#### [@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor](https://projectlombok.org/features/Constructor.html) 
	生产构造函数的注解（无参数，必须的构造函数(可以自定义构造函数的名称),全部参数的构造函数）

#### [@Data](https://projectlombok.org/features/Data.html) 
	生成全部 ToString EqualsAndHashCode, Getter Setter 无参构造函数

#### [@Value](https://projectlombok.org/features/Value.html)
	类似@Data 但是不用，详情见文档
#### [@Builder](https://projectlombok.org/features/Builder.html)
	生成建造者模式
#### [@SneakyThrows](https://projectlombok.org/features/SneakyThrows.html)
	生成抛出异常的代码
#### [@Synchronized](https://projectlombok.org/features/Synchronized.html) 
	同步代码块，锁为当前对象
#### [@Getter(lazy=true)](https://projectlombok.org/features/GetterLazy.html)
	 第一次调用时生成一次，之后会被缓存起来，代码中已加锁。
#### [@Log](https://projectlombok.org/features/Log.html)
	 生成日志代码。各种日志的配置

### 1. [lombok官网](https://projectlombok.org/index.html)
### 2. [lombok Github 地址](https://github.com/rzwitserloot/lombok)
### 3. [maven gradle等依赖地址](https://projectlombok.org/mavenrepo/index.html)
#### 3.1 maven

```
<dependencies>
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.16.10</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```
#### 3.2 Gradle配置

If your gradle version is >= 2.12 you can use lombok by adding the following to your build.gradle in the dependencies block:
	
```
compileOnly "org.projectlombok:lombok:1.16.10"
```
If you use an older version you can still use the following:

```
provided "org.projectlombok:lombok:1.16.10"
```
	