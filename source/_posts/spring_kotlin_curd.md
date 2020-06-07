---
title: 使用 Spring + kotlin 实现 CURD 应用
categories: Kotlin
tags:
- Kotlin
- Spring
date: 2017/11/25
---


在 10 小时前，我还对`kotlin`、`spring-boot`以及`gradle`不是很了解，但现在已经知道该如何使用它们运行一个简单的`CURD`应用了（或者说玩具更合适？）

虽然网上已经有很多比我写得更好的教程，不过自己写一遍能够加深印象，并且从初学者的角度记录，所以这篇笔记适合和我一样，之前没有接触过`java`、`spring`等等这些后端相关知识的人，或许能够有所收获。

<!--more-->

## 项目准备

- 可以上网的电脑（最好能翻墙）
- 配置好了`java`环境
- 安装了`IDEA`（虽然可以不用，但最好有）
- Docker（或者安装好了`Mysql`数据库并启动）

## 初始化项目

在 http://start.spring.io/ 下载脚手架，`Group`默认为`com.example`，`Artifact`改成`kotlinDemo`，这里一致后面代码就可以直接拷贝。

![初始化项目](http://upload-images.jianshu.io/upload_images/3531509-089aad43ccc44285.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击`Generate Project`后会下载一个压缩包，将压缩包解压到文件夹，使用`IDEA`打开该文件夹，文件目录应该是这样的：

```
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    ├── main
    │   ├── kotlin
    │   │   └── com
    │   │       └── example
    │   │           └── koltinDemo
    │   │               └── KoltinDemoApplication.kt
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── kotlin
            └── com
                └── example
                    └── koltinDemo
                        └── KoltinDemoApplicationTests.kt
```

打开后会出现该界面：
![import Project](http://upload-images.jianshu.io/upload_images/3531509-7a85e723bc4bf059.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

表示识别为一个`Gradle`项目，点击“OK”即可。

然后`IDEA`会下载依赖，等待下载即可。
> 如果不成功，翻墙后重试；最难下载的应该是`gradle`这个包；如果不使用`IDEA`，使用命令行进入项目根目录，运行`./gradlew build`。

### 启动项目
下载成功后，先将`build.gradle`中`dependencies`两个依赖先暂时注释：
```
dependencies {
    // compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('org.springframework.boot:spring-boot-starter-web')
    compile("org.jetbrains.kotlin:kotlin-stdlib-jre8:${kotlinVersion}")
    compile("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")
    // runtime('mysql:mysql-connector-java')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```
右下角出现该提示时，表示`IDEA`识别到修改，点击“Import Changes”。
![import changes](http://upload-images.jianshu.io/upload_images/3531509-e0a5cb1d63f1b1ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> `spring-boot-starter-data-jpa`和`spring-boot-starter-data-jpa`是用来连接数据库的，因为暂时还没有数据库直接运行会无法启动。

注释之后，就可以点击右上角的运行按钮来启动项目。
![运行项目](http://upload-images.jianshu.io/upload_images/3531509-1a260ddc68a64f7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当看到下方控制台打印了如下信息，并且没有报错，就表示运行成功：

![成功启动服务器](http://upload-images.jianshu.io/upload_images/3531509-c81b4887ff957c8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 不使用`IDEA`则在项目根目录下使用命令`./gradlew bootRun`运行项目。

如果出现该错误提示：

```
Cannot determine embedded database driver class for database type NONE
```

![没有配置数据库](http://upload-images.jianshu.io/upload_images/3531509-946bcfc0b12c336a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

则表示需要暂时注释和数据库有关的依赖，或者在`resource/application.properties`中配置数据库信息。

运行成功后，打开浏览器`http://127.0.0.1:8080`，可以看到如下页面：

![index page](http://upload-images.jianshu.io/upload_images/3531509-e29ee4252e9d226d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

虽然显示`Error`，但成功表示我们运行起了服务器，只是还没有配置该显示什么内容。

如果是如下内容：

![not hava server](http://upload-images.jianshu.io/upload_images/3531509-9acd889841b0e566.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就表示根本没有服务器在运行，检查`IDEA`下面控制台是否和上面成功的截图一样，重新打开浏览器查看。

## 第一个页面

在`kotlinDemo`文件夹下，新增`Controller.kt`文件，内容如下：

```
package com.example.koltinDemo

import org.springframework.web.bind.annotation.*

@RestController
class Controller {
    @RequestMapping("/")
    fun get() = "Hello Spring and kotlin"
}
```

如果使用`IDEA`，则更加方便

![创建文件](http://upload-images.jianshu.io/upload_images/3531509-10b77a6e2cf42585.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而且只需要在新生成的文件内，添加：

```
@RestController
class Controller {
    @RequestMapping("/")
    fun get() = "Hello Spring and kotlin"
}
```
然后点击`@RestController`，按下键盘`option + enter`，如果是`Windows`应该是`ctrl + enter`？就会自动添加`import org.springframework.web.bind.annotation.*`这一行。

OK，到此我们第一个页面就写好了，重新运行项目（停止并启动）。刷新页面（如果还没有关闭的话，否则就再次打开`127.0.0.1:8080`），Wow！页面出现了“Hello Spring and kotlin”，而不是之前让人讨厌的`Error`字眼。

![第一个页面](http://upload-images.jianshu.io/upload_images/3531509-819f85e83a233db5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 连接数据库

好了，令人激动的环节到了，这也是折腾了我最久的环节，虽然现在感觉`So easy`。

### 初始化数据库

我们使用`Docker`来运行一个数据库供我们使用，很简单：

```bash
docker run --name store_db -d -e MYSQL_ROOT_PASSWORD=123 -e MYSQL_DATABASE=store -p 3306:3306 mysql:latest 
```

然后查看下是否启动成功：

```bash
docker ps -a
```
如果出现
![启动 mysql ](http://upload-images.jianshu.io/upload_images/3531509-8f2cb34a646b52f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

则表示创建`mysql`数据库成功，
> 同时需要注意`STATUS`必须为`Up`，且`PORTS`必须为`0.0.0.0:3306->3306/tcp`。

OK，接下来就直接连接数据库并向其插入数据吧！

### 配置数据库信息
还记得之前注释的数据库相关依赖吗，将其取消注释（注意Import Changes）：
```
dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('org.springframework.boot:spring-boot-starter-web')
    compile("org.jetbrains.kotlin:kotlin-stdlib-jre8:${kotlinVersion}")
    compile("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")
    runtime('mysql:mysql-connector-java')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```
然后在`resource/application.properties`中添加如下内容：

```
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/store?autoReconnect=true&useSSL=false
spring.datasource.username=root
spring.datasource.password=123
# Hibernate ddl auto (create, create-drop, update)
spring.jpa.hibernate.ddl-auto = create-drop
```
重新启动项目，刷新页面，和之前一样？哦，那真是太好了，没有错误就是好消息。

### 创建实体

实体是什么，用来做什么？额，大概就是类似于约定数据库中的表会有什么样的字段，以及如何操作数据库的东西？

增加`Entity.kt`文件，添加如下内容：

```
package com.example.koltinDemo

import javax.persistence.*

@Entity
@Table(name = "user")
data class Customer(
        var firstName: String = "",
        var lastName: String = "",

        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        var id: Long = 0
)
```

这将表示我们会在`store`数据库中，有一张`user`表，表会有`id`、`firstName`和`lastName`字段。

然后再增加`Interface.kt`文件，内容如下：

```
package com.example.koltinDemo

import org.springframework.data.jpa.repository.JpaRepository


interface StoreRepository : JpaRepository<Customer, Long> {
}
```

### 插入数据库

然后将`Controller.kt`改写为如下内容：

```
package com.example.koltinDemo

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.annotation.*

@RestController
class CustomerController
    @Autowired constructor(val repository: StoreRepository) {

    @RequestMapping("/")
    fun findAll() = repository.findAll()

    @RequestMapping("/create", method = arrayOf(RequestMethod.POST))
    @ResponseBody
    fun create(@RequestBody customer: Customer): Customer = repository.save(customer)
}
```

OK，重启服务，再次刷新页面。页面上没有可爱的`Hello Spring`了，而是变成了`[]`，但这更可爱好吗，这表示我们从数据中读取数据了！虽然数据库中还是空的。

所以我们要向数据库中插入数据，如果你有`Postman`那就简单多了，没有也没事，我们使用命令行来实现。

在命令行输入如下内容：

```bash
curl -i -H "Content-Type: application/json" -X POST \
-d '{"firstName": "Hello", "lastName": "Spring framework"}' \
http://localhost:8080/create
```
> `{"firstName": "Hello", "lastName": "Spring framework"}`就是我们要发送的数据。

如果显示

```bash
HTTP/1.1 200
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sat, 25 Nov 2017 07:10:14 GMT

{"firstName":"Hello","lastName":"Spring framework","id":1}%
```

就表示成功了！再次刷新我们的浏览器页面看看，有数据了！！

万一没有，检查是否有`HTTP/1.1 200`字样，如果没有就用`Postman`试试看，如下：

![使用Postman 插入数据](http://upload-images.jianshu.io/upload_images/3531509-4ba895ab4023d3c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样是表示成功，重点是那个`200 OK`。

![插入数据成功](http://upload-images.jianshu.io/upload_images/3531509-52fcd3a5137c2c52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 查找数据

当然现在也可以算实现了查找，只是查找所有的数据而已，接下来我们实现根据`lastName`查找数据。

`Interface.kt`改写为如下：

```
interface StoreRepository : JpaRepository<Customer, Long> {
    fun findByLastName(name: String): List<Customer>
}
```
`Controller`变成这样：
```
class CustomerController
    @Autowired constructor(val repository: StoreRepository) {

    @RequestMapping("/")
    fun findAll() = repository.findAll()

    @RequestMapping("/create", method = arrayOf(RequestMethod.POST))
    @ResponseBody
    fun create(@RequestBody customer: Customer): Customer = repository.save(customer)
    // 这里是新增的
    @RequestMapping("/{name}")
    fun findByLastName(@PathVariable name: String) = repository.findByLastName(name)
}
```

重启服务，先随便插入几条不同的数据，访问首页看看效果。

```json
[
  {
    "firstName": "Hello",
    "lastName": "Kotlin",
    "id": 1
  },
  {
    "firstName": "Hello",
    "lastName": "World",
    "id": 2
  },
  {
    "firstName": "Hello",
    "lastName": "ltaoo",
    "id": 3
  },
  {
    "firstName": "Hello",
    "lastName": "Spring-boot",
    "id": 4
  }
]
```

插入的数据，现在我们要查询`lastName`为`ltaoo`的记录，看是否存在。在地址栏后面增加`/ltaoo`，地址变成：`http://127.0.0.1:8080/ltaoo`，回车，页面只显示一条记录了，而且正是`lastName`为`ltaoo`的那条，表示查询成功。

### 更新数据

`Controller.kt`增加如下代码，就按照之前添加代码的方式位置一样。

```
@PutMapping("/update")
fun updateUser(@RequestBody user: Customer) {
    repository.save(user)
}
```

重启服务，同样先随便插入几条数据，然后用命令行或者`Postman`发送如下数据：

```
curl -i -H "Content-Type: application/json" -X PUT \                                                                              ltaoo@ltaoodeMacBook-Pro
-d '{"firstName": "UPdate", "lastName": "Data", "id": 1}' \
http://localhost:8080/update
```
> 表示发送的数据为`{"firstName": "UPdate", "lastName": "Data", "id": 1}`，且这次发送的方式为`PUT`，之前插入数据是用`POST`。除了数字 1 之外，都要用双引号包裹。


这表示要将`id`为 1 的这条数据，`firstName`修改为`Update`，`lastName`修改为`Data`，回车后显示`200`，就表示成功， 刷新页面也能看到新修改的数据。

### 删除数据

同样是`Controller.kt`，增加如下代码：

```
@DeleteMapping("/del/{id}")
@ResponseBody
fun deleteEmployee(@PathVariable id: Long) {
    repository.delete(id)
}
```
重启服务，先增加两条记录，然后使用命令行发送如下命令：

```bash
curl -i -H "Content-Type: application/json" -X DELETE \                                                                           ltaoo@ltaoodeMacBook-Pro
http://localhost:8080/del/1
```
> 表示已`DELETE`方式请求`http://localhost:8080/del/1`地址。从这里可以看到增删改查对应了四种请求方式`POST`、`DELETE`、`PUT`、`GET`。

然后刷新浏览器页面查看所有数据，会发现原本两条数据只剩一条了，表示删除成功。

## 总结

到此，就完成了一个能够实现向数据库增删改查的`Web`应用了，全部的代码加起来可能都不超过两百行，这就是`Spring-boot`的强大之处吧。

当然这个应用很简陋，甚至都算不上应用，只是一个玩具，但这是作为学习`Kotlin`甚至说学习后端开发的一个起点，一个萌芽。

学习的过程就是不断搜索、学习、总结的过程，总体来说，从中我收获了：

- 了解了`gradle`是什么，用来做什么，怎么用。
- 了解了`Spring`连接数据库的方式
- 学习如何从`Post`请求中接收数据
- 知道了什么是实体
- 知道了`@RestController`、`@RequestMapping`以及`@PathVariable`这些装饰器？的作用
- 知道了一个`Spring`项目基本结构
- 知道了什么是包`package`
- 知道了在`IDEA`中如何自动导包
- 对`kotlin`语法有了一丢丢的了解

以及很多很多，收获满满，最后很感谢愿意在网上分享博客的同学。

> 最后附上打包的代码，虽然不知道能不能够直接运行，但能看是肯定的。
https://pan.baidu.com/s/1c2jNs3Y

## 遇到的问题

在过程中遇到很多问题，有自己代码写错的，也有大小写错误的，但每一个问题都值得记录下来，说不定也能给别人以启发呢。

### unresolved reference xxx

复制代码后`IDEA`提示这个，表示这个变量找不到来源，使用`option + enter`也不能自动导包，我遇到的可能有两种情况

- 该变量对应的依赖未在`build.gradle`中配置
- 该变量依赖项目中的包，但是路径配置不正确

### ApplicationEventMulticaster not initialized - call 'refresh' before multicasting events via the cont

可能是重复写了某个函数，记不太清了。

### Error creating bean with name xxx

代码写得有问题，具体上面问题不知道。。。

### Cannot determine embedded database driver class for database type NONE

没有配置数据库信息

### Warning about SSL connection when connecting to MySQL database

数据库地址后面必须加上`?autoReconnect=true&useSSL=false`如`jdbc:mysql://localhost:3306/Peoples?autoReconnect=true&useSSL=false`

## 参考
- [Kotlin + Spring Boot服务端开发](https://segmentfault.com/a/1190000007336486)
- [writing-a-restful-backend-using-kotlin-and-spring-boot](https://medium.com/@dime.kotevski/writing-a-restful-backend-using-kotlin-and-spring-boot-9f162c96e428)
- [kotlin-spring-boot-mysql-jpa-hibernate-rest-api-tutorial](https://www.callicoder.com/kotlin-spring-boot-mysql-jpa-hibernate-rest-api-tutorial/)
- [crud-operations-spring-rest-resources-kotlin](https://blog.codecentric.de/en/2017/04/crud-operations-spring-rest-resources-kotlin/)

