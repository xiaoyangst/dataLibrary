文档地址：https://gorm.io/zh_CN/docs/index.html

只简单介绍部分内容，详细可见官网。

## 连接数据库和配置日志打印sql

```go
import (
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"log"
	"os"
	"time"
)

func main() {
	// Default logger
	newLogger := logger.New(
		log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer
		logger.Config{
			SlowThreshold: time.Second, // Slow SQL threshold
			LogLevel:      logger.Info, // Log level
			Colorful:      true,        // Disable color
		},
	)

	// 连接数据库
	// 需要自行填写 账号 密码 地址+端口 数据库名
	dsn := "root:root@tcp(192.168.1.103:3306)/gormdb?charset=utf8mb4&parseTime=True&loc=Local"
	_, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
		Logger: newLogger,
	})
	if err != nil {
		panic("failed to connect database")
	} else {
		fmt.Println("connected to database")
	}

}
```

还支持相关配置，如果有需要可以从官网去看需要哪些配置，下面举例：

```go
db, err := gorm.Open(mysql.New(mysql.Config{
  DSN: "gorm:gorm@tcp(127.0.0.1:3306)/gorm?charset=utf8&parseTime=True&loc=Local", // DSN data source name
  DefaultStringSize: 256, // string 类型字段的默认长度
  DisableDatetimePrecision: true, // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
  DontSupportRenameIndex: true, // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
  DontSupportRenameColumn: true, // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
  SkipInitializeWithVersion: false, // 根据当前 MySQL 版本自动配置
}), &gorm.Config{})
```

## gorm的Model的逻辑删除



## 通过NullString解决不能更新零值的问题

在GORM中，默认情况下，字段如果是零值（比如 `""` 空字符串、`0`、`false` 等），更新操作会被忽略，不会写入数据库。这是因为GORM会自动跳过“零值字段”，防止误更新。

但有时候，**我们希望能够把字段更新成零值**，比如把字符串字段更新为空字符串 `""`。这时就需要用GORM提供的`sql.NullString`、`sql.NullInt64`等类型，或者GORM自己封装的`NullString`类型来实现。

`sql.NullString` 是Go标准库`database/sql`提供的一个结构体，包含两个字段：

```go
type NullString struct {
    String string
    Valid  bool // Valid=true表示这个值有效（非NULL），Valid=false表示NULL
}
```

GORM通过`NullString`的`Valid`字段判断是否更新。如果`Valid=true`，就会写入`String`的值；如果`Valid=false`，则写入NULL。

假设有个User表，有个`Name`字段是字符串，想更新Name为""（空字符串），代码如下：

```go
import (
    "database/sql"
    "gorm.io/gorm"
)

type User struct {
    ID   uint
    Name sql.NullString
}

func updateUserName(db *gorm.DB, userID uint, newName string) error {
    user := User{
        ID: userID,
        Name: sql.NullString{
            String: newName,
            Valid:  true,  // 表示更新这个字段，即使是空字符串也会更新
        },
    }

    return db.Model(&User{}).Where("id = ?", userID).Updates(user).Error
}
```

调用：

```go
err := updateUserName(db, 1, "")
if err != nil {
    panic(err)
}
```

这时，`Name`字段会被更新成数据库中的空字符串 `""`，而不会被GORM忽略。

------

你可以用GORM自带的`gorm.io/datatypes`包中的 `NullString`，功能类似：

```go
import "gorm.io/datatypes"

type User struct {
    Name datatypes.NullString
}
```

## 软删除

GORM 的 **软删除（Soft Delete）** 是一个内置功能，用于在不真正删除数据库记录的前提下，标记记录为“已删除”。这样数据还在数据库里，但在查询时会被自动排除。

GORM 通过在模型中加一个名为 `DeletedAt` 的字段，并用 GORM 的 `gorm.DeletedAt` 类型来实现软删除：

```go
import (
    "gorm.io/gorm"
)

type User struct {
    ID        uint
    Name      string
    DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

只要模型中包含 `gorm.DeletedAt` 字段，GORM 就会启用软删除机制。

### 1. 删除数据（软删除）

```go
db.Delete(&user)
```

等价于执行 SQL：

```sql
UPDATE users SET deleted_at = '2025-05-31 12:34:56' WHERE id = ?
```

### 2. 查询数据（自动排除软删除的记录）

```go
db.Find(&users)
```

GORM 默认会自动添加：

```sql
WHERE deleted_at IS NULL
```

所以你不会查到已“删除”的记录。

### 3. 查出被软删除的记录（包含所有记录）

```go
db.Unscoped().Find(&users)
```

这个 `.Unscoped()` 方法会取消所有 GORM 的自动条件，包括软删除。

### 4. 物理删除（永久删除）

```go
db.Unscoped().Delete(&user)
```

这个是真正的 `DELETE FROM users WHERE id = ?`。

