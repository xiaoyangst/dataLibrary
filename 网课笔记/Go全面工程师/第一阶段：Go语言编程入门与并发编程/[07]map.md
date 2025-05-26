## map 的初始化和赋值

### 初始化

（1）使用 make

```Go
m := make(map[string]int) // 创建空的 map，key 为 string，value 为 int

m := make(map[string]int, 10) // 预估容量为 10，提升性能
```

（2）使用字面量初始化

```go
m := map[string]int{
	"apple":  3,
	"banana": 5,
	"orange": 2,
}
```

（3）声明后赋值

```go
var m map[string]int // nil map，尚未分配内存
// m["key"] = 1 // ❌ panic: assignment to entry in nil map

// 必须先初始化
m = make(map[string]int)
m["key"] = 1
```

### 赋值

```go
m := make(map[string]int)

m["a"] = 1
m["b"] = 2
m["c"] = 3
```



## map 进行 for 循环遍历

```go
for key, value := range m {
	fmt.Printf("%s -> %d\n", key, value)
}
```

`map` 的 key 必须是可比较类型（如字符串、整数、结构体等，不能是切片、map、函数）。

## 判断 map 中是否存在元素和删除元素

元素是否存在：

```go
value, exists := m["a"]
if exists {
	fmt.Println("a:", value)
} else {
	fmt.Println("a 不存在")
}
```

&nbsp;

删除元素：

```go
delete(m, "a") // 删除 key 为 "a" 的键值对
```

