# Scala的使用

简介

```
Scala是将面向对象和面向函数式整合在一起,基于JVM的编程语言，支持Java的所有语法和特性，又高于Java
```

下载配置

```
步骤一：
	在Scala官网https://www.scala-lang.org/download/2.11.8.html，这里使用2.11.8（需要跟jdk适配），这里下载Windows版本scala-2.11.8.zip，API帮助文档scala-docs-2.11.8.zip，源码包scala-sources-2.11.8.tar.gz
步骤二：
	①解压scala-2.11.8.zip到D:\盘下，增加系统环境变量SCALA_HOME追加到系统环境变量path中
	②解压scala-docs-2.11.8.zip中的api到D:\scala-2.11.8\doc下
	③解压scala-sources-2.11.8.tar.gz到D:\scala-2.11.8\lib下
步骤三：验证
	window命令窗口使用命令：scala，查看scala版本
步骤四：idea配置scala
	idea默认不支持scala开发（天然支持java开发），需要额外安装jetbrain提供的scala插件
	①启动Idea，在File -> Settings -> Plugins在线安装Scala插件，之后重新启动idea
	②开发时，需要手动在项目 -> 右键 -> Add Frameworks Support -> 勾选Scala
	③进一步的关联源码
```

### 语法

##### 标识符的命名规则

```
基本命名规则与java类似，不同之处：
	①首字符可以为操作符(+-*/),但是必须至少连续两个操作符，比如
		var ++ = 10
        var -/// = 9
        但是不提倡使用
    ②使用··包裹的任意字符都可以作为变量，比如
    	var `true` = 99
```

##### 关键字

```
package,import,class,object,trait(代替java中的接口和抽象类),extends,with,type,forSome
private,protected,abstract,sealed,final,implicit,lazy（惰性函数，类似java的懒加载）,override
try,catch,finally,throw
if,else,match,case,do,while,for,return,yield
def,val（类似java的final）,var
this,super
new
true,false,null
```

##### 运算符

```
与java不同之处：
	①取消++，用+=代替
	②取消--，用-=代替
	③取消三元运算符，用if else代替，比如
		var i = if(逻辑表达式) 2 else 1
	④{}给变量赋值，比如
		var j = {
			if(逻辑表达式) 1 else 2
		}
```

##### 数据类型

```
Unit（类似java void）
Int，Double,Float,Char,Byte，Long，Short，Boolean
引用类型
```

##### 变量

```
两种声明方式：
	①与java相同，根据字面量进行类型推断
		val | var i = 1
	②与java不同，显示声明类型并赋值
		val | var i: 类型 = 值
```

##### 条件

```
与java的不同之处：
	①if-else，所有的分支都有返回值，默认返回()，比如
		var j = {
			if(1 > 2) 1
		}
		输出j:()
	②如果if-else{}中的逻辑语句只有一行，{}可以省略
	③如果if-else{}中的逻辑语句只有一行，且所属方法有返回值，{}和return都可以省略
```

##### 循环

```
①对比java的增强for，不同之处：
	for(迭代的变量 <- 集合/数组)
	
	或者：
	for(i <- 1 until 100)，注意：[start,end)
	
	或者：
	for(i <- 1 to 100)，注意：[start,end]
	
②对比java的普通for，不同之处：
	for(i <- 1 to 100 if i % 2 == 0)，只要满足逻辑表达式才进入循环体
	
	或者：
	for(i <- 1 to 100;j = i + 1)
	
③嵌套for循环，不同之处：
	for(i <- 1 to 10;j <- 1 to 5)
	
④循环返回值,将每次循环的结果放到Vector中，最后返回Vector
	var res = for(i <- 1 to 100) yield {
		if(i % 2 == 0)
			i
		else
			"不是偶数"
	}
⑤控制步长
	for(i <- Range(1,100,3)),步长是3
	
⑥多重for
	for(i <- 集合/数组;j <- 集合/数组)
```

##### 函数

```
①声明函数的区别：
	def 函数名(变量1: 类型1,变量2: 类型2 = 缺省值, ...): 返回类型 = {
		函数体
	}
	
	注意：
		①返回类型可以是：Unit（类似java void），Int，Double,Float,Char,Byte，Long，Short，Boolean，以及引用类型
		②参数可以带缺省值

②调用函数的区别：
	函数名(值1，值2, ...)，会覆盖缺省值
	
	或者
	函数名(变量1:类型1 = 值1)，只覆盖变量1，其他使用缺省值
	
③函数形参的区别：
	函数中形参默认被val修饰，不可再次赋值
④可变参数的区别：
	def 函数名(变量1: 类型1,变量2: 类型*): 返回类型 = {
		函数体
	}
	注意：变量2是参数集合
⑤惰性函数（在第一次使用时才会执行方法体），被lazy修饰
	lazy val res = sum(1,2) //先不执行
	println(res) //调用时执行sum(1,2)
	注意lazy所修饰的变量只能是val类型
```

##### 异常

```
①不检查编译期异常，只有运行时异常
②语法：
	 try{
	 
	 }catch{
	 	case ex: 异常类型1 => {
	 	
	 	}
	 	case ex: 异常类型2 => {
	 	
	 	}
	 }finally{
	 
	 }
```

##### 类定义

```
scala中没有构造器重载，但是用辅助构造器替换
声明方式一，默认无参构造函数：
	[修饰符] class 类名{
		[修饰符] var | val 属性名 : 类型 = 初值
	}
声明方式二，自定义构造函数：
	[修饰符] class 类名(参数1:类型1,参数2:类型2, ...){
		@BeanProperty
		[修饰符] var | val 属性名1 = 参数1
		[修饰符] var | val 属性名2 = 参数2
		[修饰符] var | val 属性名3 : 类型 = 初值
		
		//多个辅助构造器
		def this(参数1:类型1){
			this(值1,值2,...) //必须写，调用主构造器
			this.属性名1 = 参数1
		}
		
		def this(参数2:类型1){
			this(值1,值2,...) //必须写，调用主构造器
			this.属性名2 = 参数2
		}
	}
	

	不同之处：
		①一个.scala文件中可以定义多个public访问类型的类
		②不能显示声明public，否则报错
		③属性必须显式声明，不能使用类型推断
		④使用_，让系统按照类型分配默认值，以下对应类型的默认值
			#############################################
			类型					_对应的值
			#############################################
			Byte Short Int Long		0
			Float Double 			0.0
			String 和引用类型		 null
			Boolean					false
			#############################################
		⑤通过注解@BeanProperty，为属性生成get/set方法
		⑥类/属性的访问级别只有三种（默认是public）：public， protected（本类和子类访问），private（本类访问）
```

##### 包

```
在同一个scala文件中可以有多个包，每个包都可以创建类。语法：
	package 包1{
		package 包2{}
		package 包3{}
	}
```

##### 数组

```
声明方法：
	定长数组：Array(元素1，元素2，...)
	可变数组：A
```

