* **go语言入门提纲**
  * [数组](#数组) (`easy` )

## 数组
1、**创建数组的几种方式**
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
