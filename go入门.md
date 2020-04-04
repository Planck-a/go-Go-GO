* **go语言入门提纲**
  * [常见占位符](#常见占位符) (`必背` )
  * [数组](#数组) (`easy` )
  * [特殊函数](#特殊函数) (`rand` )
  * [切片](#切片)
  * [string](#string)
  
## 数组
1、**数组初始化的四种方式**

未初始化的数组，数值类型默认`0`，布尔类型默认为`false`
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
   fmt.Println(“arr2[%d][%d]=%v",i,j,v1)    
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
2、**浮点数**
```
%b
```
3、**指针地址**
```
Printf("%p", &people)   //十六进制地址，输出0x4f57f0
```
4、**布尔占位符**

5、**字符型** ：%c

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

切片是通过指针地址访问`底层数组`的，底层数组是一个长度固定的数组，通过扩容机制实现动态数组的功效。相当于C++中vector的扩容机制。

内置函数：`append()、len() 和 cap() `
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
