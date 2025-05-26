Go 语言标准库提供了一个包：

```go
import "container/list"
```

这个 `list` 是一个 **双向链表（Doubly Linked List）**，支持前插、后插、删除、移动等操作：

| 功能             | 方法或操作                               | 描述                                    |
| ---------------- | ---------------------------------------- | --------------------------------------- |
| **创建链表**     | `list.New()`                             | 创建一个新的双向链表                    |
| **添加元素**     | `PushFront(v)`                           | 在头部添加元素                          |
|                  | `PushBack(v)`                            | 在尾部添加元素                          |
| **插入元素**     | `InsertBefore(v, e)`                     | 在元素 `e` 前插入元素 `v`               |
|                  | `InsertAfter(v, e)`                      | 在元素 `e` 后插入元素 `v`               |
| **删除元素**     | `Remove(e)`                              | 删除元素 `e`（`*list.Element`）         |
| **访问首尾元素** | `Front()` / `Back()`                     | 获取首/尾元素指针（`*Element`）         |
| **遍历链表**     | `e := l.Front(); e != nil; e = e.Next()` | 正向遍历（从前往后）                    |
|                  | `e := l.Back(); e != nil; e = e.Prev()`  | 反向遍历（从后往前）                    |
| **移动元素位置** | `MoveToFront(e)`                         | 将元素 `e` 移到链表头部                 |
|                  | `MoveToBack(e)`                          | 将元素 `e` 移到链表尾部                 |
|                  | `MoveBefore(e, mark)`                    | 将元素 `e` 移到 `mark` 前面             |
|                  | `MoveAfter(e, mark)`                     | 将元素 `e` 移到 `mark` 后面             |
| **链表长度**     | `Len()`                                  | 返回链表中元素个数                      |
| **元素值访问**   | `e.Value`                                | 获取元素 `e` 中存储的值（类型为 `any`） |