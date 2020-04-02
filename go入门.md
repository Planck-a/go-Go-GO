* **go语言入门提纲**
  * [常见占位符](#常见占位符) (`必背` )
  * [数组](#数组) (`easy` )

## 数组
1、**数组地址问题**
```
package main
import("fmt")

func main(){
  var a[5]int
  fmt.Println(a)
  fmt.Println(&a)  //数组的地址
  fmt.Println(&a[0])  //首元素的地址，等于&a
  fmt.Println(&a[1])  //第二个元素的地址，相当于&a[0]+8，go语言int默认是int64，占8字节
}
```
## 常见占位符
1、**整数占位符**
```
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
