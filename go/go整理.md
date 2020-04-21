

go-proxy配置：(https://goproxy.io/)  
go env -w GO111MODULE=on
go env -w GOPROXY="https://goproxy.io,direct"

go env -w GO111MODULE=on
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct

编译运行  go build/install xxx.go  
格式化： gofmt -w xxx.go (-w写入文件)  

## 自定义包  
需要包外面访问的函数和变量都需要大写开头
https://gowalker.org/  
获取package文档：
go doc fmt Printf

## struct  
new(tructxxx) 等价于 &strunctxxx{}  
Go语言的结构体嵌套：
		1.模拟继承性：is - a
			type A struct{
				field
			}
			type B struct{
				A //匿名字段
			}

		2.模拟聚合关系：has - a
			type C struct{
				field
			}
			type D struct{
				c C //聚合关系
			}


## 方法  
	方法：method	一个方法就是一个包含了接受者的函数，接受者可以是命名类型或者结构体类型的一个值或者是一个指针。
		所有给定类型的方法属于该类型的方法集
	语法：
		func （接受者） 方法名(参数列表)(返回值列表){
		}
	总结：method，同函数类似，区别需要有接受者。(也就是调用者)
	对比函数：
		A：意义
			方法：某个类别的行为功能，需要指定的接受者调用
			函数：一段独立功能的代码，可以直接调用
		B：语法
			方法：方法名可以相同，只要接受者不同
			函数：命名不能冲突

## 接口  
### 接口：interface
		在Go中，接口是一组方法签名。
		当某个类型为这个接口中的所有方法提供了方法的实现，它被称为实现接口。
		Go语言中，接口和类型的实现关系，是非侵入式
		//其他语言中，要显示的定义
		class Mouse implements USB{}
	1.当需要接口类型的对象时，可以使用任意实现类对象代替
	2.接口对象不能访问实现类中的属性

### 多态：一个事物的多种形态
		go语言通过接口模拟多态
		就一个接口的实现
			1.看成实现本身的类型，能够访问实现类中的属性和方法
			2.看成是对应的接口类型，那就只能够访问接口中的方法
	接口的用法：
		1.一个函数如果接受接口类型作为参数，那么实际上可以传入该接口的任意实现类型对象作为参数。
		2.定义一个类型为接口类型，实际上可以赋值为任意实现类的对象

### 空接口(interface{})  
		不包含任何的方法，正因为如此，所有的类型都实现了空接口，因此空接口可以存储任意类型的数值。

	fmt包下的Print系列函数：
		func Print(a ...interface{}) (n int, err error)
		func Printf(format string, a ...interface{}) (n int, err error)
		func Println(a ...interface{}) (n int, err error)

### 接口断言：  
		方式一：
			1.instance := 接口对象.(实际类型) //不安全，会panic()
			2.instance, ok := 接口对象.(实际类型) //安全

		方式二：switch
			switch instance := 接口对象.(type){
			case 实际类型1:
					....
			case 实际类型2:
					....
			....
			}

### type：用于类型定义和类型别名

		1.类型定义：type 类型名 Type
		2.类型别名：type  类型名 = Type


## 错误error  
error：
内置的数据类型，内置的接口
	定义方法：Error() string

使用go语言提供好的包：
	errors包下的函数：New()，创建一个error对象
	fmt包下的Errorf()函数：
		func Errorf(format string, a ...interface{}) error


## 包package  
### 关于包的使用：  
	1.一个目录下的统计文件归属一个包。package的声明要一致
	2.package声明的包和对应的目录名可以不一致。但习惯上还是写成一致的
	3.包可以嵌套
	4.同包下的函数不需要导入包，可以直接使用
	5.main包，main()函数所在的包，其他的包不能使用
	6.导入包的时候，路径要从src下开始写
	 */
	//utils.Count()
	//timeutils.PrintTime()
	//
	//pk1.MyTest1()
	//
	//utils.MyTest2()
	//
	//fmt.Println("---------------------")
	//p1 :=p.Person{Name:"王二狗",Sex:"男"}
	//fmt.Println(p1.Name,p1.Sex)


	/*
	init()函数和main()函数
	1.这两个函数都是go语言中的保留函数。init()用于初始化信息，main()用于作为程序的入口
	2.这两个函数定义的时候：不能有参数，返回值。只能由go程序自动调用，不能被引用。
	3.init()函数可以定义在任意的包中，可以有多个。main()函数只能在main包下，并且只能有一个。
	4.执行顺序
		A：先执行init()函数，后执行main()函数
		B：对于同一个go文件中，调用顺序是从上到下的，也就是说，先写的先被执行，后写的后被执行
		C：对于同一个包下，将文件名按照字符串进行排序，之后顺序调用各个文件中init()函数
		D：对于不同包下，
			如果不存在依赖，按照main包中import的顺序来调用对应包中init()函数
			如果存在依赖，最后被依赖的最先被初始化
				导入顺序：main-->A-->B-->C
				执行顺序：C-->B-->A-->main

	5.存在依赖的包之间，不能循环导入
	6.一个包可以被其他多个包import，但是只能被初始化一次。
	7._操作，其实是引入该包，而不直接使用包里面的函数，仅仅是调用了该包里的init()
	 */
	//utils.Count()

	//pk1.MyTest1()

	//database/sql

	//db,err :=sql.Open("mysql","root:hanru1314@tcp(127.0.0.1:3306)/my1802?charset=utf8")
	//if err != nil{
	//	fmt.Println("错误信息：",err)
	//	return
	//}
	//fmt.Println("链接成功：",db)

### 常用包  
* time  

* OS: IO  

## 并发编程  
进程 线程 协程
并行和并发
goroutine 轻量级线程(协程)    
两级线程模型：GPM

### runtime  

go run -race xxx.go


## 反射机制  


## 切片  

## make和new  
```go
type Foo map[string]string
type Bar struct {
    itemOne string
    itemTwo int
}
//正确赋值方法
aa := new(Bar)  //不能用make
(*aa).itemOne = "hello"

bb := make(Foo) //不能用new
bb["name"] = "jack"


```


## testing 包  


## map实现set, 工厂模式
go语言没有set


## 接口interface  
type 接口类型名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
}  
sort包  
sort.StringSlice/IntSlice/Float64Slice  sort.Sort(xxx)


# FAQ  
https://blog.csdn.net/stpeace/article/details/81675028  
1. 简短声明的变量只能在函数内部使用  
```go
   // 错误示例
myvar := 1	// syntax error: non-declaration statement outside function body
func main() {
}
 
// 正确示例
var  myvar = 1
func main() {
}
```
2. 使用简短声明来重复声明变量  
不能用简短声明方式来单独为一个变量重复声明， := 左侧至少有一个新变量，才允许多变量的重复声明：

3. struct 的变量字段不能使用 := 来赋值以使用预定义的变量来避免解决：  
```go
// 错误示例
type info struct {
	result int
}
 
func work() (int, error) {
	return 3, nil
}
 
func main() {
	var data info
	data.result, err := work()	// error: non-name data.result on left side of :=
	fmt.Printf("info: %+v\n", data)
}
 
 
// 正确示例
func main() {
	var data info
	var err error	// err 需要预声明
 
	data.result, err = work()
	if err != nil {
		fmt.Println(err)
		return
	}
 
	fmt.Printf("info: %+v\n", data)
}
```



GO Proxy:  
export GOPROXY=https://mirrors.aliyun.com/goproxy/


Go jira:
https://javascript.ctolib.com/go-jira.html
https://github.com/andygrunwald/go-jira


# go ldap 
https://blog.csdn.net/weixin_39594447/article/details/87804225 
https://mojotv.cn/2019/07/15/ldap
