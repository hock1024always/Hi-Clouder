# Gorm

## 概述信息

**中文文档：** https://gorm.io/zh_CN/docs/models.html

### 用途

1. 规范一致代码工整
2. 减少一定的工作量
3. 对于一些通用的系统部署更方便
4. 解耦数据库与数据访问层更加方便

### 缺点

1. 数据访问层并不能因为使用`orm`而显著减小
2. 大量使用反射，导致程序性能不佳
3. `orm`提供大量表关系接口

### 入库操作

1. 指定表或者model
2. 正向选择或者反向选择那些字段入库
3. 单挑记录还是批量记录，批量可以指定每批次处理记录条数

### 数据查询操作

1. 指定表或者model
2. 正向选择或者反向选择那些字段查询
3. `where`子句构建（and，or，in，out）查询条件
4. `orderby`子句
5. limit子句
6. group having子句
7. limit offset子句
8. 结果填充，填充到对象，集合，或切片

### 更新

1. 指定表或者model
2. 正向选择或者反向选择那些字段更新
3. whrere子句

### 删除

1. 指定表或者model
2. where子句

### 事务

1. 开启事务
2. 提交事务
3. 回滚事务
4. 事务嵌套
5. 事务回调
6. 事务钩子
7. 事务隔离级别
8. 事务表
9. 事务表回滚
10. 事务表提交  

## 具体使用

### 创建数据库结构体

```go
import (
	"database/sql"
	"gorm.io/gorm"
)

type Teacher struct {
	gorm.Model
	Name      string `gorm:"size: 256"`
	Email     string `gorm:"uniqueIndex"`
	Age       uint   `gorm:"check:age>30"`
	WorkYears uint8
	Birthday  int64 `gorm:"serializer:unixtime;type:time"`
	StuNumber sql.NullString
	Roles     Roles `gorm:"serializer:json"`
	JobInfo   Job   `gorm:"embeddedPrefix:job_"`
	JobInfo2  Job   `gorm:"type:bytes;serializer:gob"`
}

type Job struct {
	Title    string
	Location string
}
type Roles []string
```

### 创建数据库连接

```go
var DB *gorm.DB
var dsn = "root:123456@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"

func init() {
	var err error
	DB, err = gorm.Open(mysql.New(mysql.Config{
		DSN:               dsn, // DSN data source name
		DefaultStringSize: 256, // string 类型字段的默认长度
	}), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
	})
	if err != nil {
		log.Fatal(err)
	}
	setPool(DB)
}
func setPool(db *gorm.DB) {
	sqlDB, err := db.DB()
	if err != nil {
		log.Fatal(err)
	}
	sqlDB.SetMaxIdleConns(10)
	sqlDB.SetMaxOpenConns(5)
	sqlDB.SetConnMaxLifetime(time.Hour)
}
```

#### **dns**

1. `用户名:密码@协议(地址:端口)/数据库名?参数1=值1&参数2=值2...`
2. **用户名和密码**
   `root:123456`
   - `root` - 数据库用户名
   - `123456` - 数据库密码
3. **网络协议和地址**
   `@tcp(127.0.0.1:3306)`
   - `tcp` - 使用 TCP 协议连接
   - `127.0.0.1` - MySQL 服务器地址（这里是本地）
   - `3306` - MySQL 默认端口号
4. **数据库名称**
   `/test`
   - 要连接的数据库名称是 `test`
5. **连接参数**（以 `?` 开始，多个参数用 `&` 分隔）
   `?charset=utf8mb4&parseTime=True&loc=Local`
   - `charset=utf8mb4` - 设置字符集为 UTF-8（支持完整的 Unicode，包括 emoji）
   - `parseTime=True` - 将数据库中的时间类型（如 `DATE`, `DATETIME`）解析为 Go 的 `time.Time` 类型
   - `loc=Local` - 使用本地时区处理时间
6. **其他可选参数**
   - `timeout=5s` - 连接超时时间（如 5 秒）
   - `readTimeout=30s` - 读超时时间
   - `writeTimeout=30s` - 写超时时间
   - `allowNativePasswords=true` - 允许 MySQL 原生密码认证

#### **数据库连接池**

```go
func setPool(db *gorm.DB) {
	sqlDB, err := db.DB()//返回数据库连接对象的指针
	if err != nil {
		log.Fatal(err)
	}
	sqlDB.SetMaxIdleConns(10)//最大空闲连接数，最多可以保持10个空闲连接
	sqlDB.SetMaxOpenConns(5)//最多并发处理量，最多课同时处理5个数据库连接
	sqlDB.SetConnMaxLifetime(time.Hour)//最大生命周期，不使用的话一小时之后自动关闭
}
```

#### **初始化函数**

记下来就行

```go
func init() {
	var err error
	DB, err = gorm.Open(mysql.New(mysql.Config{
		DSN:               dsn, // DSN data source name
		DefaultStringSize: 256, // string 类型字段的默认长度
	}), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
	})
	if err != nil {
		log.Fatal(err)
	}
	setPool(DB)
}
```

### 数据库迁移

```go
func init() {
	DB.AutoMigrate(&Teacher{})
	//DB.Migrator().
}
```

`AutoMigrate()` 是 GORM 提供的**自动建表方法**：

- 如果数据库中没有 `Teacher` 表，则**自动创建**。
- 如果表已存在但结构发生变化（如新增字段），则**自动修改表结构**（添加列、修改类型等）。

`Migrator()` 是 GORM 提供的**底层表操作接口**，可以更精细地控制迁移行为（如检查表是否存在、删除表等）

**作用**

1. **自动管理数据库表结构**
   - 无需手动执行 SQL 语句（如 `CREATE TABLE`）。
   - 适合快速迭代开发，但生产环境可能需要更谨慎的迁移策略（如使用 [迁移工具](https://gorm.io/docs/migration.html)）。
2. **关联模型**
   - 如果 `Teacher` 关联其他模型（如 `Student`），GORM 会自动处理外键关系。
3. **注意点**
   - **不会删除未使用的字段**：如果从 `Teacher` 结构体中删除某个字段，`AutoMigrate()` 默认不会删除表中的列（需手动处理）。
   - **生产环境慎用**：频繁修改表结构可能导致数据丢失，建议配合版本控制工具（如 [Goose](https://github.com/pressly/goose) 或 [Flyway](https://flywaydb.org/)）。

### 数据库创建

下面代码向数据库写一条记录

```go
var teacherTemp = Teacher{
	Name:      "test",
	Email:     "test@test.com",
	Age:       30,
	WorkYears: 10,
	Birthday:  time.Now().Unix(),
	StuNumber: struct {
		String string
		Valid  bool
	}{String: "10", Valid: true},
	Roles: []string{"普通用户", "讲师"},
	JobInfo: Job{
		Title:    "teacher1",
		Location: "beijing",
	},
	JobInfo2: Job{
		Title:    "teacher2",
		Location: "beijing",
	},
}

func CreateRecord() {
	t := teacherTemp
	res := DB.Create(&t)
	if res.Error != nil {
		log.Println(res.Error)
		return
	}
	println(res.RowsAffected, res.Error, t)
	//正向选择
	t1 := teacherTemp
	res = DB.Select("Name", "Age").Create(&t1)//直插入这两个字段
	println(res.RowsAffected, res.Error, t1)
	//反向选择
	t2 := teacherTemp
	res = DB.Omit("Email", "Birthday").Create(&t2)//除了这两个字段都要插入
	println(res.RowsAffected, res.Error, t2)

	//批量传入
	var teachers1 = []Teacher{{Name: "king", Age: 40}, {Name: "jack", Age: 41}, {Name: "nick", Age: 35}}
	DB.CreateInBatches(teachers1, 2)//每批插入2个，在这里第一批插入2个、第二批插入1个
	for _, t := range teachers1 {
		println(t.ID)
	}
}
```

### 数据库查找

```go
func Query() {
	//指定表或者model
	//查询单条数据
	//查询第一条
	t := Teacher{}
	res := DB.First(&t)//res中没有数据、只有一些参数信息；数据会直接修改变量t
	println(res.RowsAffected, res.Error, t)

	//查询最后一条
	t = Teacher{}
	res = DB.Last(&t)
	println(res.RowsAffected, res.Error, t)

	//无排序，取第一条
	t = Teacher{}
	res = DB.Take(&t)//数据库显示展示的第一条
	println(res.RowsAffected, res.Error, t)
    
	//将结果填充到集合不支持特殊类型处理，无法完成类型转换
	result := map[string]interface{}{}
	res = DB.Model(&Teacher{}).Omit("Birthday", "Roles", "JobInfo2").First(&result)
	println(res.RowsAffected, res.Error, result)

	result = map[string]interface{}{}
	res = DB.Table("teachers").Omit("Birthday", "Roles", "JobInfo2").First(&result)
	println(res.RowsAffected, res.Error, result)

	result = map[string]interface{}{}
	res = DB.Table("teachers").Take(&result)
	println(res.RowsAffected, res.Error, result)

	var teachers []Teacher
	res = DB.Where("name=?", "张三").Or("name=?", "king").Order("id desc").Limit(10).Find(&teachers)
	println(res.RowsAffected, res.Error, teachers)
}
```

### 事务

#### 默认禁用事务

这样做可以一定程度上提高性能

```go
// 全局禁用
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  SkipDefaultTransaction: true,
})

// 持续会话模式
tx := db.Session(&Session{SkipDefaultTransaction: true})
tx.First(&user, 1)
tx.Find(&users)
tx.Model(&user).Update("Age", 18)
```

#### 整体回滚与部分回滚

```go
func Transaction() {
	t := teacherTemp
	t1 := teacherTemp
	t2 := teacherTemp
	t3 := teacherTemp

	DB.Transaction(func(tx *gorm.DB) error {
		// 在事务中执行一些数据库操作（比如创建、更新、删除等）
		if err := tx.Create(&t).Error; err != nil {
			return err
		}
		
        //回滚子事务，不影响大事务最终结果
		tx.Transaction(func(tx1 *gorm.DB) error {
			tx1.Create(t1)
			return errors.New("rollback t1")//这个报错返回会在gorm中被捕获，不会导致主事务的回滚
		})

        
        //嵌套事务
		tx.Transaction(func(tx2 *gorm.DB) error {
			if err := tx2.Create(&t2).Error; err != nil {
				return err//会导致主事务的回滚
			}
			return nil
		})
		
        if err := tx.Create(&t3).Error; err != nil {
			return err
		}
		return nil
	})
}
```

1. 最外层的 `DB.Transaction` 开始一个主事务
2. 首先创建记录 `t` - 如果失败会返回错误并导致整体回滚
3. 然后开始第一个嵌套事务（子事务1）创建 `t1` 并故意返回错误：
   - 这个错误会导致子事务1回滚
   - 但由于子事务的错误被捕获在子事务内部，不会传播到主事务
4. 接着开始第二个嵌套事务（子事务2）创建 `t2`：
   - 如果成功，这部分会提交（在GORM中嵌套事务实际上是保存点）
   - 如果失败，会返回错误导致主事务回滚
5. 最后创建 `t3` - 如果失败会返回错误并导致整体回滚