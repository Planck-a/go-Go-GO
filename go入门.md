* **go语言入门提纲**
  * [常见占位符](#常见占位符) (`必背` )
  * [数组](#数组) (`easy` )

## 数组
1、**数组初始化的四种方式**
```
var a[3]int = [3]int {1,2,3}
var a = [3]int {1,2,3}
var a = [...]int {1,2,3}
var a = [...]string {1:"love",0:"I",2:"U"}
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
