* **go语言入门提纲**
  * [常见占位符](#常见占位符) (`必背` )
  * [数组](#数组) (`easy` )
  * [特殊函数](#特殊函数) (`rand` )
  * [切片](#切片)
  
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
1、**创建一个切片**

内置函数：`append()、len() 和 cap() `
```
s1 := make([]int, 5)   //其长度和容量都是 5 个元素
s2 :=make([]int,3,5)   //其长度为 3 个元素，容量为 5 个元素
s3 := make([]int, 5, 3) //报错，长度>容量会报错

s4 :=[]string{"Jack", "Mark", "Nick"} //通过字面量创建切片,其长度和容量都是 3 个元素

var myNum []int  //创建nil切片，比如，函数要求返回一个切片但是发生异常的时候，就会返回一个nil切片

```
