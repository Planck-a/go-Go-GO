* **go语言入门提纲**
  * [常见占位符](#常见占位符) (`必背` )
  * [循环控制](#循环控制)
  * [数组](#数组) (`easy` )
  * [特殊函数](#特殊函数) (`rand` )
  * [切片](#切片)
  * [string](#string)
  * [map](#map)(`哪些不能作key`)
  * [面向对象编程](#面向对象编程)(`重点`)
  * [普通函数和成员函数方法的区别](#普通函数和成员函数方法的区别)
  * [接口interface](#接口interface)
  * [类型断言assert](#类型断言assert)(`判断传入参数的类型` `多态数组的遍历`)
  * [反射reflection](#反射reflection)
  * [文件操作](#文件操作)
  * [协程goroutine](#协程goroutine)
  * [异常处理机制](#异常处理机制)
  
## 入门语法
* 从控制台输入
```
fmt.Scanf("%d\n",&key)
fmt.Scanf("%s\n",&userPwd)
```
* 带判断的死循环
```
for flag{ 

}
```
  
## 异常处理机制
* 子协程panic时希望主线程不受影响，通过defer+recover来实现
```
package main

import (
	"fmt"
	"time"
)

func sayHello(){
	for i:=0;i<10;i++{
		time.Sleep(time.Second)
		fmt.Println("hello world")
	}
}

func test(){
	//这里可以使用defer + recover来解决
	defer func(){
		if err := recover(); err != nil{
			fmt.Println("协程发生错误",err)
		}
	}()

	var mymap map[int]string
	mymap[0]="golang"
}

func main(){
	go sayHello()
	go test()

	time.Sleep(time.Second*5)


}
```
## 协程goroutine
1、**获取计算机的CPU数目**
```
import(
	"fmt"
	"runtime"
)
func main(){
	cupNum := runtime.NumCPU()
	fmt.Println("CUP数目=",cupNum)

	//设置程序实际运行中使用的CPU数目
	runtime.GOMAXPROCS(cupNum-1) 
}
```
2、**开启协程示例**
* c/c++的线程是内核态的，比较耗资源；go语言的协程是逻辑态的，相比起线程会更轻便，可以轻松开启上万个协程
* go语言中协程的工作模式为MPG：M是主线程  P是协程执行需要的上下文  G是协程
```
func test(){
	for i :=0;i<10;i++{
		fmt.Println("test () hello,world "+strconv.Itoa(i))
		time.Sleep(time.Second)
	}
}

func main(){
	go test() //开启了协程
	for i :=0;i<10;i++{
		fmt.Println("main () hello,world "+strconv.Itoa(i))
		time.Sleep(time.Second)
	}
	//注意：以主线程为准！主线程一旦执行完毕，即使协程没完成，程序也结束
}
```
3、**协程中的互斥锁**
* 需求：计算1-200 的各个数的阶乘，并把结果放在全局变量map中
* 利用全局的互斥锁sync.Mutex，实现安全地共享
```
package main

import (
	"fmt"
	"time"
	"sync" //提供了基本的同步工具，如互斥锁
)
var (
	myMap = make(map[int]int,10)   
	lock sync.Mutex
)
func cal(n int){
	res :=1
	for i:=1;i<=n ;i++{
		res *= i
	}
	lock.Lock()
	myMap[n] = res
	lock.Unlock()
}
func main(){
	//开启多个协程完成任务
	for i:=1;i<= 200; i++ {
		go cal(i)
	}
	time.Sleep(time.Second * 10)
	lock.Lock()
	for i:=1;i<= 200; i++ {
		fmt.Printf("map[%v]=%v\n",i,myMap[i])
	}
	lock.Unlock()
}
```
4、**管道介绍**
* channel是线程安全的，操纵方式和队列一样，FIFO。管道不会自动扩容
* 管道是有类型的，string类型的管道只能放string变量
* 如果管道既要存int又要存string，而可以定义为空接口类型的。但是只读取时也必须类型断言地读，比较麻烦:var intchan chan interface{}
```
package main
import "fmt"	

func main(){

	//创建存放三个int地管道,管道不会自动增长。当容量满了之后， 除非取出才能再插入
	var intchan chan int
	intchan = make(chan int,3)

	fmt.Printf("%v\n",intchan)  //intchan是个引用，会打印出地址

	intchan <- 10
	num := 2020
	intchan <- num

	fmt.Printf("intchan len = %v  , cap = %v \n",len(intchan),cap(intchan)) 

	//从队头取出元素,放在num2
	num2 := <-intchan
	fmt.Println(num2)
	fmt.Printf("intchan len = %v  , cap = %v \n",len(intchan),cap(intchan))  

	//从队头取出元素，丢弃不用
	<-intchan
	fmt.Printf("intchan len = %v  , cap = %v \n",len(intchan),cap(intchan)) 

}
```
5、**interface{}管道**
```
package main

import(
	"fmt"
)

type Cat struct{
	Name string
	Age int
}

func main(){
	
	allchan := make(chan interface{},3)
	allchan <- 10
	allchan <-"Jack"
	cat := Cat{"小花猫",5}
	allchan <-cat

	//如果希望获取管道中第三个元素，必须先将前两个推出去
	<-allchan
	<-allchan
	newCat := <-allchan //从管道中取出来的是interface{}

	fmt.Printf("newcat = %T,newcat = %v\n",newCat,newCat) //可以顺利打印出结构体变量

	//fmt.Printf("newCat.Name = %v",newCat.Name),这种方法错误，编译通不过

	//空接口不能直接取出结构体成员变量的值，必须先用类型断言转换为结构体变量
	a:=newCat.(Cat)
	fmt.Printf("newCat.Name = %v",a.Name)
}
```
6、**多协程求素数**
* 管道会有死锁问题，有个管道容量为5，如果程序判断只有写没有读，且写入的数据量>5，就会报错；但是只要有人在读管道，那么就不用担心死锁问题
* 管道的特性：如果不关闭管道就读取，会因为阻塞而导致deaklock；close管道后，只能读，不能写
* 管道是线程安全的，可以用来实现对全局变量的互斥访问

```
package main

import (
	"fmt"
	"time"
)

func putNum(intchan chan int){
	for i:=1;i<=80000;i++{
		intchan <- i    //虽然管道只有1000容量，但是只要有人在读管道，那么就不用担心死锁问题
	}
	close(intchan)
}

func isprime(intchan chan int,primechan chan int,exitchan chan bool){
	var flag bool
	for{
		num,ok := <- intchan
		if !ok{
			//intchan 取不出东西了
			break
		}
		flag = true
		for i:=2;i< num ;i++{
			if num % i ==0 {
				//说明不是素数
				flag = false
				break
			}
		}
		if(flag){
			primechan <- num
		}
	}
	fmt.Println("有一个协程因为取不出数据而关闭")
	//此时不能直接关闭primechan
	exitchan<- true
}

func main(){
	start := time.Now().Unix()
	intchan := make(chan int,1000)
	primechan := make(chan int,20000)   //放入素数
	//标识管道退出
	exitchan := make(chan bool,4)

	go putNum(intchan)
	go isprime(intchan,primechan,exitchan)
	go isprime(intchan,primechan,exitchan)
	go isprime(intchan,primechan,exitchan)
	go isprime(intchan,primechan,exitchan)
	var end int64
	go func (){
		for i:=0;i<4;i++{
			<- exitchan
		}
		//当取出四个结果，就可以放心地关闭管道 primechan
		close(primechan)
		end = time.Now().Unix()
	}()
	count :=1
	for{
		_,ok := <- primechan
		if !ok{
			break
		}
		//fmt.Printf("第%v个素数=%v \n",count,res)
		count++

	}
	fmt.Println("协程消耗地时间为：",end-start)

}
```
7、**管道的使用细节**
* 01 子协程panic时希望主线程不受影响，通过defer+recover来实现
```
package main

import (
	"fmt"
	"time"
)

//如果只是某个协程发生错误，通过defer+recover来解决
//不会影响主线程

func sayHello(){
	for i:=0;i<10;i++{
		time.Sleep(time.Second)
		fmt.Println("hello world")
	}
}

func test(){
	//这里可以使用defer + recover来解决
	defer func(){
		if err := recover(); err != nil{
			fmt.Println("协程发生错误",err)
		}
	}()

	var mymap map[int]string
	mymap[0]="golang"
}

func main(){
	go sayHello()
	go test()

	time.Sleep(time.Second*5)


}
```
* 02 使用select解决从管道取数据阻塞的问题
	* 传统的方法在遍历管道时，如果不关闭管道就读取，会因为阻塞而导致deaklock；所以要先close，后读取
	* 但是如果实际开发不确定什么时候关闭管道，可以用select解决
```
package main
import (
	"fmt"
	"time"
)
func main(){
	//使用select解决从管道取数据阻塞的问题

	intchan := make(chan int,10)
	for i:=0;i<10;i++{
		intchan <- i
	}

	stringchan := make(chan string , 5)
	for i:=0;i < 5;i++{
		stringchan <- "hello"+fmt.Sprintf("%d",i)
	}

	//传统的方法在遍历管道时，如果不关闭管道就读取，会因为阻塞而导致deaklock
	//如果实际开发不确定什么时候关闭管道，用select解决
	for{
		select{
			//注意：如果这里取不到，会自动跳到下个case
			case v:= <- intchan:
				fmt.Printf("intchan读到的数据为：%d\n",v)
				time.Sleep(time.Second)
			case v:= <- stringchan:
				fmt.Printf("stringchan读到的数据为：%s\n",v)
				time.Sleep(time.Second)
			default:
				fmt.Println("都读不到数据")
				time.Sleep(time.Second)
				break
		}
	}
}
```
* 03 只读和只写管道
```
package main
import "fmt"	
func main(){
	//声明为只写，类型仍然是chan int，只是某个属性修改
	var chan2 chan<- int   
	chan2 = make (chan int,3)
	chan2 <- 1
	
	//声明为只读
	var chan3 <-chan int
	num2 ：= <-chan3
}
```

## 文件操作
1、**os.File 结构体**`
```
func Open(name string)(file *File,err error)
func (f *File) Close() error
 ``` 
  

## 循环控制
1、switch-case
```
func main(){
	for i,v := range arr{
		switch v{
			case "1":
			
			case "2":
			
			default:
				
		}
	}
}
```
2、for循环
* 死循环
```
for{

}
```
  
## 数组
1、**数组初始化的四种方式**

未初始化的数组，数值类型默认`0`，布尔类型默认为`false`。
```
var a [3]int = [3]int {1,2,3}
var a = [3]int {1,2,3}
var a = [...]int {1,2,3}
var a = [...]string {1:"love",0:"I",2:"U"}

var a[]int   //这种方式是定义切片，即动态数组
```
或者可以省略var，写成：
```
a [3]int ：= [3]int {1,2,3}
a ：= [3]int {1,2,3}
a ：= [...]int {1,2,3}
a ：= [...]string {1:"love",0:"I",2:"U"}
```
2、**数组地址问题**
```
package main
import("fmt")

func main(){
  var a [5]int
  for i:=0;i < len(a);i++{
     fmt.Println("请输入第%d个元素"，i+1)
     fmt.Scanln(&a[i])  //从键盘接收字符，存到数组a中
  }
  fmt.Println(a)
  fmt.Println(&a)  //数组的地址
  fmt.Println(&a[0])  //首元素的地址，等于&a
  fmt.Println(&a[1])  //第二个元素的地址，相当于&a[0]+8，go语言int默认是int64，占8字节
}
```
3、**数组遍历for-range**

数组遍历的时候，下标从0开始，越界会报panic
```
var arr [3]int = [3]int {1,2,3}
for index,value := range arr{
   //不需要index时，可以用_替代
   //index,value是局部遍历，for循环外不可用
   //index,value名称不固定，可自行定义
}
```
4、**数组做函数参数进行传递**
* 数组是值传递，如果需要传地址，应写成*[3]int
```
func test01(a [3]int) //go语言中认为 [3]int 和 [4]int是两种不同的数据类型
{
   a[0]=1
}
func test02(a *[3]int){//指针做函数参数，可以改变形参指向地址空间中的内容
   (*a)[0]=1
}
func main(){
  a:=[3]int{0,0,2}
  test01(a)//数组是值传递，所以函数中和主函数中是两份地址空间，互不影响
  fmt.Println(a)
  
  test02(&a)
  fmt.Println(a)
}
```
**形参传递时几种常见的错误：**
```
//数组做形参传递时要严格对应
[3]int -------->[]int   编译出错
[3]int--------->[4]int  编译出错
```
5、**byte类型数组**
```
func main(){
 var a [26]byte
 for index,_ ：= range a{
   a[i]='A'+byte(i)  //必须是相同的数据类型之间才可以进行加减。'A'+'1' ===>'B','A'+1会报错
 }
}
```
6、**二维数组**
```
 arr2 := [2][3]int{{1,2,3},{4,5,6}}
 fmt.Println(arr2)   //打印数组，结果为[1,2,3][4,5,6]
 fmt.Println(arr2[0]) //打印第一行,结果为[1,2,3]
 fmt.Println(&arr2)  //数组首地址，数值上&arr2 == &arr2[0] == &arr2[0][0]
 
 for i,v := range(arr2){
  for j,v1 := range(v){
   fmt.Println(“arr2[%d][%d]=%v\t",i,j,v1)    
  }
 }
```


## 常见占位符
**通用占位符**
```
Printf("%v",1)    //自动识别数据类型，输出1
```
1、**整数占位符**
```
Printf("%d",1)    //十进制，输出1
Printf("%b", 5)  //二进制，输出101
Printf("%q", 0x4E2D)	 //单引号围绕，输出'中'
```
2、**浮点数**：%b

3、**指针地址**：%p
```
Printf("%p", &people)   //十六进制地址，输出0x4f57f0
```
4、**布尔占位符**

5、**字符型** ：%c

6、**类型**：%T

## 特殊函数
1、**rand函数**
```
import(
  "math/rand"
  "time"
)
func main(){
 var a [5]int
 rand.seed(time().Now().Unix())  //为了让每次生成的随机数都不同，先要设置随机种子。Unix()单位秒，UnixNano单位纳秒
 //rand.seed(time().Now().UnixNano())    生成的随机数更为分散
 for index,_ :=arrange a{
   a[index]=rand.Intn(100)   //生成 0<= x <100 的随机数
 }
}
```
## 切片
1、**初始化一个切片**

切片三要素：`类型(指针地址)、长度和容量`

* 切片是引用类型

* 切片是通过指针地址访问`底层数组`的，底层数组是一个长度固定的数组，通过扩容机制实现动态数组的功效。相当于C++中vector的扩容机制。
* 内置函数：`append()、len() 和 cap() `
* 只声明是不分配内存的，用make创建才是分配内存后的。`切片make后才可以使用`
```
s1 := make([]int, 5)   //其长度和容量都是 5 个元素
s2 :=make([]int,3,5)   //其长度为 3 个元素，容量为 5 个元素
s3 := make([]int, 5, 3) //报错，长度>容量会报错

s4 :=[]string{"Jack", "Mark", "Nick"} //通过字面量创建切片,其长度和容量都是 3 个元素

var myNum []int  //创建nil切片，比如，函数要求返回一个切片但是发生异常的时候，就会返回一个nil切片

```
2、**通过一个切片创建另一个切片**
```
s1 := []in{1，2，3，4，5}
newsli := sl[1:3:4] 
```
* 表示从下标为i处开始切；切片的长度(j-i)，选取元素是`有左无右`；控制切片的容量(k-i)。如果没有给定 k，则表示切到底层数组的最尾部
* newsli切片和s1切片`共享底层数组`，若newsli[0]=5，则对应s1[1]变为5了
* 如果二次切片的容量>底层数组容量，报错panic: runtime error: slice bounds out of range

3、**追加元素append()**
* append() 函数会返回一个包含修改结果的`新切片`，底层数组不一定改变了。底层数组什么时候会变呢？？什么时候会发生分离呢？？
* 如果要修改当前数组的内容，那么还需要用这个数组来接收返回值
* 函数 append() 总是会增加新切片的长度，而`容量有可能会改变`，也可能不会改变，这取决于被操作的切片的可用容量
```
myNum := []int{10, 20, 30, 40, 50}
N1 := append(myNum,100,200,300) //myNum内容没有发生变化

myNum = append(myNum,100,200,300) //如果需要原数组变化，就必须还是用这个数组来接收返回值
```
 
```
myNum := []int{10, 20, 30, 40, 50}
newNum := myNum[1:3]// 创建新的切片，其长度为 2 个元素，容量为 4 个元素
newNum = append(newNum, 60)// 使用原有的容量来分配一个新元素,指为60
```
* 函数 append() 总是会增加新切片的长度，而`容量有可能会改变`，也可能不会改变，这取决于被操作的切片的可用容量。
* 由于底层数组未发生变化，所以仍然和原始的切片共享同一个底层数组，myNum 中索引为 3 的元素的值也被改动了。
* 如果切片的底层数组没有足够的可用容量，append() 函数会`创建一个新的底层数组`，与原数组分离。扩容机制为：在切片的容量小于 1000 个元素时，总是会成倍地增加容量。一旦元素个数超过 1000，容量的增长因子会设为 1.25

4、**小心二次切片的append()操作**
```
例1：
fruit := []string{"Apple","Orange","Plum","Banana","Grape"}
myFruit := fruit[2:3:4]   //从fruit下标为2开始，长度为1，容量为2的二次切片。相当于只包含了Plum
myFruit = append(myFruit, "Kiwi") //此时对二次切片append操作，而二次切片尚且有空闲容量，所以append结果随机且容易出错

正确做法：
myFruit := fruit[2:3:3]  //二次切片创建时，令长度==容量
myFruit = append(myFruit, "Kiwi") //此时append()的话，会直接创建一个新的底层数组，与原有的底层数组分离
```
5、**将一个切片追加到另一个切片**
```
num1 := []int{1, 2}
num2 := []int{3, 4}
fmt.Printf("%v\n", append(num1, num2...))  // 将两个切片追加在一起，并显示结果

num2 = append(num2,num2...)  //num2追加到自身
```
* 底层实现机制是怎样的？

6、**切片之间的拷贝操作copy()**

把切片 src 中的元素拷贝到切片 dst 中，返回值为拷贝成功的元素个数。如果 src 比 dst 长，就截断；如果 src 比 dst 短，则只拷贝 src 那部分。
```
num1 := []int{10, 20, 30}
num2 := make([]int, 5)
count := copy(num2, num1)  //目标地址为num2，源地址为num1
fmt.Println(count)    //输出3
fmt.Println(num2)     //输出[10 20 30 0 0]
```
7、**切片做函数参数**
* 在64位架构机器上，一个切片需要 24 字节的内存：指针字段需要 8 字节，长度和容量字段分别需要 8 字节。
* 切片做函数参数进行传递时，仅仅传递24字节的信息。效率更高，等同于传数组指针
```
func test(sli []int){
  sli[0]=555   //会影响原切片中的值
}
func main(){
  a :=[]int{1,2,3}
  test(a)
}
```
## string
1、string底层
* 底层是切片，所以支持slice相关操作。
* 不能直接通过索引s[2]='A'来修改字符串。如果要修改，需先进行string--->[]byte或者[]rune，然后再进行编辑，再转成string
```
s1:="wodojwd"
a1 := []byte(s1)
a1[2]='A'
s1 = string(a1)
```
* 转为byte可以修改英文和数字，但是不能改变中文。因为中文占3字节，所以中文的话应该转为rune，兼容汉字

2、string的拼接:fmt.Sprintf
```
	var de string
	fmt.Scanfln(&money)
	balance+=money //余额balance
	fmt.Scanfln(&note)
	de += fmt.Sprintf("\n收入\t%v\t%v\t%v",balance,money,note)
```

## map
1、**key可以是什么类型，不能是什么类型**
* key可以是`数值`，`string`，布尔，指针，channel；也可以是这几种类型的接口、结构体和数组
* key不可以是slice、map、function，因为这几种数据类型没法进行比较
* value可以是各种类型，常用的是数值、string、map、struct
* 若value是空接口interface{}，那么这个value可以接收任意类型的数据

2、**创建一个map**
* 声明一个map
```
 var a map[string]string
 var a map[int]string
 var map[string]map[string]string   //value仍然是个map
```
* 声明一个map并不会分配内存，初始化需要用`make`，分配内存后才可以复制并使用
* 分配了内存之后，比如通过make分配了3，当插入第4组key-value时，会自动扩容
```
 var a map[int]string
 a[1]="woaini"   //会报错panic:...is nil
 
 //1
 var a map[int]string
 a=make(map[int]string,10)  //通过make分配大小为10对key-value的空间
 //2  推荐
 a := make(map[int]string)
 a[0]="wdaw"
 a[1]="fsdw"
 //3
a :=map[int]string{
 1:"abc",
 2:"fds"，//最后一组key-value后的","必不可少
} 
a[5]="adsds"//后续可以直接插入
```
3、**删除、更新、查询**
```
// 删除，key不存在则啥也不干
delete(m, "name")

// 更新
m["name"] = "咖啡色的羊驼2"

// 查询，key不存在返回value类型的零值
i := m["name"] // 三种查询方式，
i, ok := m["name"]
_, ok := m["name"]
```
4、**遍历**
 * key是无序的，与C++不同。在遍历的时候并不会按照你传入的顺序
 ```
 for k, v := range m { 
    fmt.Println(k, v)
}
 ```
 * 若要强制进行有序遍历，则需要先把key放在数值中进行排序
 ```
 import "sort"
var keys []string
// 把key单独抽取出来，放在数组中
for k, _ := range m {
    keys = append(keys, k)
}
// 进行数组的排序
sort.Strings(keys)  //int类型用 sort.Ints()
// 遍历数组就是有序的了
for _, k := range keys {
    fmt.Println(k, m[k])
}
 ```
5、**map作函数参数进行传递**
* Golang中函数参数是没有引用传递的，均为值传递。也即是说传递的是数据的拷贝。
* map本身就是个引用，指向的是一个数据结构(包括元素个数、扩容常量、数组指针地址等成员变量)
* map作为形参或返回参数的时候，传递的是值(map变量的地址)的拷贝，即使发生扩容也不会改变这个地址。发生改变的map结构体中某个成员变量的值
```
var m map[int64]int64
m = make(map[int64]int64, 1)
fmt.Printf("m 原始的地址是：%p\n", m)  //m 原始地址是：0xc42007a180
changeM(m)
fmt.Printf("m 改变后地址是：%p\n", m)//m 改变后地址是：0xc42007a180
fmt.Println("m 长度是", len(m))  //长度变为5
fmt.Println("m 参数是", m)     //打印map[3:2 4:2 0:2 1:2 2:2]

// 改变map的函数
func changeM(m map[int64]int64) {
	fmt.Printf("m 函数开始时地址是：%p\n", m) //m 函数开始时地址是：0xc42007a180
	var max = 5
	for i := 0; i < max; i++ {
		m[int64(i)] = 2
	}
	fmt.Printf("m 在函数返回前地址是：%p\n", m) //m 在函数返回前地址是：0xc42007a180
}

```

6、**vaule为空接口的实例**

* value为interface{}，所以可以是任意类型的数据
```
func testMap(){
	var a map[string]interface{}
	//使用map需要make
	 a = make(map[string]interface{})  
	 a["name"]="红孩儿"
	 a["age"] = 30
	 a["address"] = "洪崖洞"

	 data, err :=json.Marshal(&a)
	 if err != nil{
		fmt.Printf("序列化失败，err= %v",err)
	}
	fmt.Printf("序列化后的结果为：%v",string(data))
}
```

## 面向对象编程
1、**特点**
* 去掉了传统OOP语言的继承、方法重载、构造和析构函数、隐藏this指针
* 用struct实现面向对象特性
* 仍然具有封装、继承、多态特性，只是实现方式不同

2、**创建一个结构体**
* 声明了变量即分配了内存，不需要make
```
type cat struct{
    Name string    //字段大写开头，表示可以被其他包所引用
    Age int
    Color string
}
func main(){
    var cat1 cat
    cat1.Name="小王"
    cat1.Age=5
    
    cat2 := cat{"小张"，10}
    
    cat3 := cat{
    	Name:"小谢"，
	age:5，
    }
}
```
* go中结构体是值类型，同理作函数参数传递时也是传值。要获取地址要用&cat1
```
var cat1 cat
fmt.Println("cat1的地址是：%p"，&cat1)

var cat_ptr=&cat{"小张"，10}
```
* 创建一个结构体指针变量
```
var cat1 cat
cat1.Name="小王"
var cat_ptr *cat=cat1
```
* 获取变量中成员函数，用cat1.变量 或则 cat_ptr.变量
* 变量不能设置访问权限，相当于都是C++的public

3、**结构体变量中字段的使用注意事项**
* 结构体变量中若有map和切片，仍要遵从先make，再使用
* 结构体变量在内存中是连续分布的。如果有两个连续的字段都是`指针类型`，那么这`两个指针的地址连续`，但是两个指针指向的地方不一定连续
* 字段可以加tag，场景是序列化和反序列时。比如现在类中变量的Name，是个公有变量。但是我在传递给客户端时，了解到客户端希望接收的小写的name，那么在不破坏公有属性的前提下，可以加tag
```
type Monster struct{
	Name string'json:"name"'   //附加一个tag，那么在json时可以自动转换为小写的
	Age int'json:"age"'
	Skill string'json:"skill"'
}
func main(){
	mon1 := Monster{"牛魔王"，100，"芭蕉扇"}
	jsonstr,_= json.Marshall(mon1) //Marshall返回 ([]byte,int)
	fmt.Println(string(jsonstr)) //复习，[]byte转字符串函数，string()
	//打印出就是name:"牛魔王"，age：100,skill:"芭蕉扇"
}
```

4、**结构体默认的拷贝构造函数为值拷贝**
```
var cat1 cat
cat1.Name="小王"
cat1.Age=5

cat2 := cat1//默认是值拷贝
cat2.Name= "二狗" //因此，对cat2做修改不会影响cat1中的值

```
5、**结构体的方法**
* 结构体的方法和成员变量相同，大写公有，小写私有
* 通过接收器来定义某个结构体的方法，`接收体不一定是struct类型，也可以是基本数据类型,比如int`
```
func (p person) Setage(age int)(返回值列表){
	//这个方法只能给person这个结构体用，而且必须要加结构体变量才可以
}
```
* 给方法绑定一个结构体指针变量
```
//为了提高效率，通常方法和结构体的指针类型绑定，引用拷贝
func (p *Circle) area()float64{
	
	//按照逻辑应该这样调用，(*p).radius=20，但是实际编译器在底层有优化~
	p.radius=20  //编译器底层对结构体指针的调用进行优化，这样也不出错
	fmt.Println("地址是：%p",p) //编译器优化导致此处的p就等同于*p，所以地址和c相同
	return 3.14*p.radius*p.radius
}

func main(){
	var c Circle
	c.radius=10
	fmt.Println("地址是：%p",c)
	res2 :=c.area()
}
```
* 如果在同一个包中，或者两个包但是对公有的成员变量，可以直接p.Name来查看和修改。如果是两个包中查看私有变量，那么必须通过方法来实现
* 接收者为值类型时，可以用指针来调用；同理当接收者为指针，也可以用值变量来调用
```
type Psrson struct{
	Name string
}
func (p Person)test(){
	fmt.Println("名字是：%v",p.name)
}
func (p *Person)test02(){
	fmt.Println("名字是：%v",p.name)
}
func main(){
	p :=Psrson{"tom"}
	p.test() //正常调用
	(&p).test()//也能正常调用，语句看似引用传递，其实内部还是值传递
	
	p.test02()//正常
	(*p).test02()//正常，内其实部还是引用传递
}
```
* 如果某个类定义了string()，那么fmt.Println()就会自动调用这个方法
```
type Student struct{
	Name string
	Age int
}
func (stu *Student)String() string{
	str :=fmt.Println("name = [%v],Age=[%v]",stu.Name,stu.Age) 
	return str
}

func main(){
	stu := Student{
		Name : "Tom",
		Age : 20,
	}
	fmt.Println(&stu)  //会打印name=[Tom],Age=[20]
}
```

6、**golang没有构造函数，用工厂模式来实现**
* 场景1：在a包中定义一个student类，首字母小写导致在其他包用不了。那么如果在其他包要怎么调用呢？
* 思路就是，仿照C++里面通过定义公有函数获取私有变量
* go里面路径用 /
```
func NewStudent(n string,s int) *student{
	return &student{
		Name:n,
		age:s，
	}
}
```
7、**封装一个类**
* 场景：定义一个person私有的类，类的属性和方法均为私有。将person这个类及方法封装在model包中，供main包调用
```
package model

type  person struct{
	Name string
	age int
	sal float64
}

//工程模式，相当于构造函数
func NewPerson(name string)* person{
	return &person{
		Name:name,
	}
}
func (p *person) Setage(age int){
//p *person为一个接收器，命名时建议使用接收器类型名的第一个小写字母
//作用类似于面向对象的this
	if age>0 && age<150{
		p.age=age
	}
	else{
		fmt.Println("输入不满足要求")
		//可以在此处给默认值
	}
}
func (p *person) Getage()int{
	return p.age
}
```
* main函数中调用
```
package main
import{
	"fmt"
	"go_code/chapter1/model"
}

func main(){
	p:=model.NewPerson("smith")
	p.SetAge(18)
	age1 := p.getAge

}
```
8、**继承**
* 将多个结构体的公共部分抽象为父类，在子类结构体中嵌入父类的`匿名结构体`，便实现继承特性。同时匿名结构体可以是指针类型，这样效率更高
```
type Student struct{
	Name string
	Age int
	Score int
}
func (p *Student)Setscore(n int){
	p.Score=n
}
type Pupil struct{
	Student
	femal String
}
func (p *Pupil)tset(){
	fmt.Println("小学生写作业")
}
func main(){
	pupil :=&Pupil{}
	pupil.Student.Name="tom"  //通过匿名结构体对成员变量幅值
	pupil.Student.Age=8
	pupil.tset()
	pupil.Student.Setscore(70)
	
	//初始化的另一种方法
	pupil :=&Pupil{
		Student{"tom",8,70},
		"男"，
		}
	//如果初始化指针类型的匿名结构体
	pupil :=&Pupil{
		&Student{"tom",8,70},    //此处换成&Student{"tom",8,70}
		"男"，
		}
}
```
* 子类结构体可以使用父类`所有的字段和方法`，不论大小写
* 匿名结构体字段的访问可以简化，省略匿名结构体。编译器会处理
```
pupil.Student.Name="tom" 
pupil.Name="tom"  //此写法也是ok 的
```
* 当子类和父类中有相同的字段名字，那么在调用时采用就近原则：`先从自身找有没有定义该字段，没有的话才去匿名结构体中去找`
* 如果有同名字段，仍要访问和定义匿名结构体的字段，需要通过显示地调用 `pupil.Student.Name`
* 结构体中嵌套两个匿名结构体地话，如果两个爹中有相同的字段，且自己本身没有这个字段，那么调用两个爹中字段时必须指明调用哪个，否则报错(指示不明确)
* 结构体中嵌套有名结构体，构成组合，那么调用时必须要显式调用
```
type A struct{
	Name string
}
type B struct{
	a A
}
func main(){
	var b B
	b.Name="tom"  //报错
	b.a.Name="tom"  //正确
}
```

9、**多态**
* 多态参数和`多态数组`
```

type Usb interface{
	Start()
}

type Phone Struct{ }
func (p Phone)Start(){
 
}

type Camera Struct{ }
func (c Camera)Start(){
 
}

func main(){
	var usbArr [3]Usb  //定义多态数组，既可以存放Phone变量，也可以存放Camera变量
	usbArr[0]=Phone{}
	usbArr[1]=Phone{}
	usbArr[2]=Camera{}
}
```
* 加入断言的多态数组遍历：如果数组中变量为Phone变量，那么除了执行start()，还要执行Phone特有的call()
```
func main(){
	var usbArr [3]Usb  //定义多态数组，既可以存放Phone变量，也可以存放Camera变量
	usbArr[0]=Phone{}
	usbArr[1]=Phone{}
	usbArr[2]=Camera{}
	
	for _,v := range usbArr{
		if p2,ok ：=v.(Phone); ok{
			v.Start()
			v.Call()
		}
		else{
			v.Start()
		}
	}
}

```

## 类型断言assert:
1、 实现了接口方法的类变量可以赋值给接口，那么空接口怎么给类变量赋值？
```
type Point struct{
	x int
	y int
}
 func main(){
 	var a interface{} //空接口，可以用来接收所有的参数
	var p1 Point = Point{1,2}
	a = p1 //实现了接口方法的类变量可以赋值给接口
	var p2 Point =a  //空接口不能直接对类变量赋值
	
	var p2 Point =a.(Point)   //断言，判断a是否确实指向Point类型，如果是就转，不是就放弃
	fmt.Println(p2)
	
 }

```

2、 带判断的断言
```
type Point struct{
	x int
	y int
}
 func main(){
 	var a interface{} //空接口，可以用来接收所有的参数
	var p1 Point = Point{1,2}
	a = p1 //实现了接口方法的类变量可以赋值给接口
	var p2 Point =a  //空接口不能直接对类变量赋值
	
	p2,ok ：=a.(float64)   
	if ok == true{
		fmt.Println("转换成功")
		fmt.Println(p2)
	}
	else{
		fmt.Println("转换失败")
	}
	
	//简洁写法
	if p2,ok ：=a.(float64); ok {
		fmt.Println("转换成功")
		fmt.Println(p2)
	}
	else{
		fmt.Println("转换失败")
	}
	
 }
 ```
 3、编写函数，配合switch-case 来实现传入参数判断
 
 ```
 func TypeJudge(items...interface{}){
 	for i,v := range items{
 	switch v.(type){
		case bool :
		
		case float32:
		
		case int,int32,int64:
		
		//自定义数据类型
		case Student:
		
		
		default:
			fmt.Println("没有匹配的类型")
	}
 }
 }
 
 ```



## 接口interface
1、**定义一个接口,实现多态特性**

* 接口不能包含任何变量
* 接口中的方法`不能包含任何的方法体`：method(参数列表)返回值列表
* 场景：前人写好了一个方法，需要传入一个接口。那么后来人只用实现了接口的那几个函数，就能成功调用前人的方法

```
//定义接口，定义一种规范。高内聚低耦合
type Usb interface{
	Start()
	Stop()
}



type Phone struct{
	
}
func (p Phone)Start(){
	fmt.Println("手机工作")	
}
func (p Phone)Stop(){
	fmt.Println("手机停止工作")	
}




type Camera struct{
	
}
func (p Camera)Start(){
	fmt.Println("相机工作")	
}
func (p Camera)Stop(){
	fmt.Println("相机停止工作")	
}

type Computer struct{  }
//编写一个wirking方法，接收一个Usb类型的接口
//只要实现了接口的，即实现了接口的所有函数，就可以调用这个函数
func (c Computer)Working(u Usb){
	u.Start()
	u.Stop()
}


func main(){
	computer :=Computer{}
	phone := Phone{}
	camera :=Camera{}
	
	computer.Working(phone)
	computer.Working(camera)
}
```
2、**通用实现**
* Java中实现接口必须显示地指出实现了哪个接口，type A struct implement B。而go是基于方法的，只要实现了这个接口的所有函数，就实现了接口，不要特别指出是实现了哪个接口。

3、**sort函数**------>`重要例子`
```
type Interface interface{
	Len() int
	Less(i,j int)bool
	Swap(i,j int)
}

func sort(n Interface){
//对于这个函数而言，接收一个Interface接口，实现排序
//对传入的参数，要求必须是struct吗？？不一定，但是必须同时实现上述3个方法
}

```

4、**实现接口的不一定是struct，可以是任意类型**
```
type Interface interface{
	Start()
}

//任意类型都可以实现接口
type Inter int
func (i Inter)Start(){
	fmt.Println(" 111 ")
}

func test(Interface interface){
	Interface.Start()
}

func main(){
	var i Inter
	var in Interface = i
	in.Start()
	
	//或者通过函数调用
	test(i)
}
```
5、**一个自定义类型可以实现多个接口**

6、**hero切片的排序**
```
/*
type Interface interface{
	Len() int
	Less(i,j int)bool
	Swap(i,j int)
}*/

//1、声明hero结构体
type Hero struct{
	Name String
	Age int
}

//2、声明一个hero切片
type HeroSlice []Hero

//3、实现方法
func (hs HeroSlice) Len() int {
	return len(hs)
}
func (hs HeroSlice) Less(i,j int) bool {
	return hs[i].Age > hs[j].Age
}
func (hs HeroSlice) Swap(i,j int) {
	tmp := hs[i]
	hs[i] = hs[j]
	hs[j] = tmp
	
	//更简洁的写法
	hs[i],hs[j] = hs[j],hs[i]
}

func main(){
	var intSlice = []int{0,-1,10,7,100}
	//要求对切片进行排序1、冒泡排序   2、系统函数sort
	sort.Ints(intSlice)  //切片是引用类型，所有传变量名即可
	
	//利用sort对结构体变量进行排序
	var hs HeroSlice
	for i:=0;i<10;i++{
		hero :=Hero{
			Name : fmt.Println(rand.Intn(100)),
			Age : rand.Intn(100)
		}
		hs = append(hs,hero)
	}
	for _,v := range hs {
		fmt.Println(v)
	}
	sort.Sort(hs)
	for _,v := range hs {
		fmt.Println(v)
	}
}
```

7、**空接口**
```
func TypeJudge(items interface{}){
 	for i,v := range items{
 	switch v.(type){
		case bool :
		
		case float32:
		
		case int,int32,int64:
		
		//自定义数据类型
		case Student:
		
		
		default:
			fmt.Println("没有匹配的类型")
	}
 }
```

## 反射reflection
* reflect中interface{}，变量和reflect.Value是可以相互转换的

func Test(o interface{})        //变量----->interface{}
{
	rval := reflect.ValueOf(o)   //interface{}--->value
	ival := rval.Interface()     //vaule----->interface{}
	vval := ival.(Student)       //interface{}---->Student变量 类型断言
}

1、**对于基本数据类型**
* 获取到reflect.Type
* 获取reflect.Value
* 将 rval转成interface{},并通过断言转为需要的类型
* 传入一个int变量，如果要取出这个变量的值，必须通过 reflect.ValueOf(b).Int()

```
func reflectTest01(b interface{}){
	//通过反射获取传入的变量的 type ， kind
	//1.获取到reflect.Type
	rTyp := reflect.TypeOf(b)
	fmt.Println("rType = ",rTyp)
	fmt.Println("rType = ",rTyp.Name())

	//2.获取reflect.Value
	rVal := reflect.ValueOf(b)
	fmt.Printf("type = %T,rVal=%v\n",rVal,rVal)

	//n2:= rVal+100  rVal并不仅仅是int，所以这样加会报错
	n2:= rVal.Int()+100
	fmt.Println(n2)

	//3.将 rval转成interface{},并通过断言转为需要的类型
	iv := rVal.Interface()
	num2 := iv.(int)
	fmt.Println(num2)
	
}
```

2、**对于结构体变量**
* 获取到reflect.Type
* 获取reflect.Value
* 获取变量对应的kind
* 将 rval转成interface{},并通过断言转为需要的类型

```
type Student struct{
	Name string
	Age int
}

func reflectTest01(b interface{}){
	//通过反射获取传入的变量的 type ， kind(更大的范畴)
	//1.获取到reflect.Type
	rTyp := reflect.TypeOf(b)
	fmt.Println("rType = ",rTyp)
	fmt.Println("rType = ",rTyp.Name())

	//2.获取reflect.Value
	rVal := reflect.ValueOf(b)
	fmt.Printf("type = %T,rVal=%v\n",rVal,rVal)

	//获取变量对应的kind
	//（1）
	fmt.Printf("kind = %v",rTyp.Kind())
	//(2)
	fmt.Printf("kind = %v",rVal.Kind())

	//n2:= rVal+100  rVal并不仅仅是int，所以这样加会报错
	n2:= rVal.Int()+100
	fmt.Println(n2)

	//3.将 rval转成interface{},并通过断言转为需要的类型
	iv := rVal.Interface()
	num2 := iv.(int)
	fmt.Println(num2)
	
}
```
3、**通过reflect.ValueOf(b).Elem()对形参进行修改**
```
package main

import (
	 "fmt"
	 "reflect"
)

func reflect01(b interface{}){
	rVal := reflect.ValueOf(b)  //rVal是个指针
	fmt.Printf("类型是 = %v"，rVal.Kind())

	// rVal.SetInt(20)//panic,因为rVal是地址

	//3.rVal.Elem()获取指针指向的具体的空间
	rVal.Elem().SetInt(20) //会直接修改原地址的值
}

func main(){
	n1 := 1
	reflect01(&n1)
	fmt.Println(n1)
}
```
4、**代码判断**
```
package main

import(
	"reflect"
)
func main(){
	var str string = "tom"
	fn := reflect.ValueOf(str)
	fn.SetString("isdaa")       //false
	fn.Elem().SetString("isdaa")//true
}
```


* **获取某个Struct变量的所有字段信息**
  * reflect.Typeof(o).Field(i).Name() 和 reflect.Valueof(o) .Field(i).Interface()
  * 可以获取成员变量的名、类型、值
  * 可以获取成员函数的名、类型
  * 只能反射Struct值类型，不能反射指针类型
  
```
func Info(o interface{}){
	t := reflect.TypeOf(o)
	fmt.Println("Type:",t.Name())
	
	if k:=t.Kind();k !=reflect.Struct{
		fmt.Println("传入参数类型不正确")
		return
	}
	
	v :=reflect.ValueOf(o)
	for i:=0;i< t.NumField();i++{
		f := t.Field(i)  
		t1 := f.Name()   //字段名   reflect.Typeof(o).Field(i).Name() 
		t2 := f.Type()     //字段的数据类型  reflect.Typeof(o).Field(i).Type() 
		val := v.Field(i).Interface()  //字段的值  reflect.Valueof(o) .Field(i).Interface()
	}
	
	for i:=0;i< t.NumMethod();i++{
		m := t.Method()
		t1 := m.Name()  //方法的名字
		t2 := m.Type()   //方法的类型
	}
}
```
* **对于获取变量中匿名结构体的所有字段信息**
  *  reflect.Typeof(m).FieldByIndex([]int{0.0})
  * 获取匿名结构体成员变量的名、类型、值
  * 获取匿名结构体成员函数的名、类型
  
  ```
  type User struct{
  	Id int
	Name string
	Age int
  }
  type Manger struct{
  	User     //匿名结构体
	Title string
  }
  func main(){
  	m := Manger{
		User{1,"ok",12},
		Title: "CEO"
	}
	t:= reflect.Typeof(m)
	fmt.Println("%v\n",t.FieldByIndex([]int{0.0}))   //{0,0}对应第1个匿名结构体的第一个字段，打印出id
  }
  ```

## 普通函数和成员函数方法的区别

1、**调用方式不同**

2、**调用者和接收者的匹配程度要求不同**

* 结构体： 接收者为值类型时，可以用指针来调用；同理当接收者为指针，也可以用值变量来调用
* 函数 ：只能对应调用

```
type Psrson struct{
	Name string
}
func (p Person)test(){
	fmt.Println("名字是：%v",p.name)
}
func (p *Person)test02(){
	fmt.Println("名字是：%v",p.name)
}
func main(){
	p :=Psrson{"tom"}
	p.test() //正常调用
	(&p).test()//也能正常调用，语句看似引用传递，其实内部还是值传递
	
	p.test02()//正常
	(*p).test02()//正常，内其实部还是引用传递
}
```
