## 

### 循环变量快照问题

#### 正确的写法

```c++
package gos

import (
    "fmt"
    "testing"
)

func TestTs(t *testing.T){

    lis := []int{1,2,5}
    for _, i := range lis{
        // 匿名函数中传入循环的值，避免循环变量快照问题
        go func(a int){
            fmt.Println("a: ", a)
        }(i)
    }
}
/*
结果（因为没有做waitGroup，所以打印的个数不确定！）：
a: 1
a: 5
a: 2
*/
```

**❗️匿名函数中的循环变量快照问题：上面这个单独的变量i是被所有的匿名函数值所共享，且会被连续的循环迭代所更新的。**

**当新的goroutine开始执行字面函数时，for循环可能已经更新了f并且开始了另一轮的迭代或者（更有可能的）已经结束了整个循环，所以当这些goroutine开始读取i的值时，它们所看到的值已经是slice的最后一个元素了。**

**在匿名函数中显式地添加i这个参数（也就是形参a），我们能够确保使用的i是当go语句执行时的“当前”那个i。**

#### 错误的写法

**❗️不可以像这样写——“循环变量快照”问题：**

```c++
package gos

import (
    "fmt"
    "testing"
)

func TestTs(t *testing.T){

    lis := []int{1,2,5}
    for _, i := range lis{
        go func(){
            // 如果匿名函数中没有任何参数，i是被所有匿名函数共享的！这样i会是最后一个值
            fmt.Println("a: ", i)
        }()
    }
}
/*
结果（因为没有做waitGroup，所以打印的个数不确定！）：
a: 5
a: 5
a: 5
*/
```





