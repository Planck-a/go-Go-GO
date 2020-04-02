* **go语言入门提纲**
  * [常见占位符](#常见占位符) (`必背` )
  * [数组](#数组) (`easy` )

## 数组
1、**数组初始化的四种方式**

未初始化的数组，数值类型默认`0`，布尔类型默认为`false`
```
var a[3]int = [3]int {1,2,3}
var a = [3]int {1,2,3}
var a = [...]int {1,2,3}
var a = [...]string {1:"love",0:"I",2:"U"}

var a[]int   //这种方式是定义切片，即动态数组
```
或者可以省略var，写成：
```
a[3]int ：= [3]int {1,2,3}
a ：= [3]int {1,2,3}
a ：= [...]int {1,2,3}
a ：= [...]string {1:"love",0:"I",2:"U"}
```
2、**数组地址问题**
```
package main
import("fmt")

func main(){
  var a[5]int
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
var arr[3]int = [3]int {1,2,3}
for index,value := range arr{
   //不需要index时，可以用_替代
   //index,value是局部遍历，for循环外不可用
   //index,value名称不固定，可自行定义
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
