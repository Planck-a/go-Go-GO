* **go语言进阶提纲**
  * [go测试框架testing](#go测试框架testing) (`` )
  * [map的深入了解](#map的深入了解)


## map的深入了解
[文章](https://blog.csdn.net/u011957758/article/details/82846609)

## go测试框架testing
1、**测试流程**
* 测试文件名必须是以_test.go结尾，测试函数命名要求，TestXxx，其中Xxx的第一个字母不能是小写的，比如本例中TestAddUpper
* 形参必须是 *testing.T
* cmd上语句：go test -v
* 如果文件中有多个_test.go的测试文件，但是只想测试一个，就必须显示地调用，go test -v cal_test.go cal.go
* 只测单个函数：go test -v -test.run TestAddUpper
```cal_test.go
package cal
import "testing"  //引入一个go testing的测试框架

func TestAddUpper(t *testing.T){
	//调用
	res :=addUpper(10)
	if res != 55{
		//t.Fatalf作用相当于断言，如果出错的话就终止，并打印出日志
		t.Fatalf("addUpper()不正确，返回值=%v，期望值=%v",res,55)
	}
	t.Logf("AddUpper()正确")
}
```
```sub_test.go  
package cal
import(
	
	"testing"  //引入一个go testing的测试框架
)

//TestAddUpper测试函数命名要求，Testxxx，其中xxx的第一个字母不能是小写的
func TestGetSub(t *testing.T){
	//调用
	res :=getSub(10,5)
	if res != 5{
		//t.Fatalf作用相当于断言，如果出错的话就终止，并打印出日志
		t.Fatalf("addUpper()不正确，返回值=%v，期望值=%v",res,5)
	}

	t.Logf("TestGetSub()正确")

}
```
```cal.go
package cal

func addUpper(n int) int{
	res :=0
	for i:=1;i<=n;i++{
		res +=i
	}
	return res
}

func getSub(n1 int,n2 int) int {
	return n1-n2
}
```
2、**序列化和反序列化结果测试**
```monster_test.go
package monster
import(
	"testing"	
)

func TestStore(t *testing.T){
	monster := Monster{
		Name:"红孩儿"，
		Age :10,
		Skill : "吐火"，

	}
	res := monster.Store()
	if res != nil{
		t.Fatalf("出错了")
	}
}
```
```cal.go
package monster
import(
	"encoding/json"
	"io/ioutil"
)

type Monster struct{
	Name string 
	Age int 
	Skill string
}

func (this *Monster)Store() error {
	//序列化
	data,err :=json.Marshal(this)
	if err != nil{
		fmt.Printf("marshal err=%v",err)
		return err
	}

	//保存在文件中
	filepath := "d:/monster.txt"
	err = ioutil.WriteFile(filepath,data,0666)
	if err != nil{
		fmt.Printf("write file err=%v",err)
		return err
	}
	return nil
}

func (this *Monster)ReStore() error { 
	//读取文件
	filepath := "d:/monster.txt"
	data, err := ioutil.ReadFile(filepath,data,0666)
	if err != nil{
		fmt.Printf("read file err=%v",err)
		return err
	}

	//反序列化
	err :=json.Unmarshal([]byte(data),this)
	if err != nil{
		fmt.Printf("Unmarshal err=%v",err)
		return err
	}
	return nil
}

```
