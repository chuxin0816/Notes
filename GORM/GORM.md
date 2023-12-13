# GORM

### 安装gorm和mysql驱动

```shell
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

## 连接mysql

```go
import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
}
```

## 定义模型

```go
// gorm.Model 的定义
type Model struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
}

//模型示例
type User struct {
  gorm.Model
  Name         string
  Age          sql.NullInt64
  Birthday     *time.Time
  Email        string  `gorm:"type:varchar(100);unique_index"`
  Role         string  `gorm:"size:255"` // 设置字段大小为255
  MemberNumber *string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
  Num          int     `gorm:"AUTO_INCREMENT"` // 设置 num 为自增类型
  Address      string  `gorm:"index:addr"` // 给address字段创建名为addr的索引
  IgnoreMe     int     `gorm:"-"` // 忽略本字段
}
```

## 创建表

```go
type User struct {
	gorm.Model
	Name string `gorm:"default:'小王子'"`
	Age  uint
}

db.AutoMigrate(&User{})	//根据结构体创建表,有则更新，无则创建
```

## 创建行

```go
user:=User{Name: "李大夫",Age: 33}
db.NewRecord(user) // 主键为空返回`true`
db.Create(&user)   // 创建user
db.NewRecord(user) // 创建`user`后返回`false`

//当某个字段有默认值时传入空串会使用默认值
//解决办法：将字段修改为*string，传入时用new(string)
```

## 查询

### 原生SQL

```go
// 执行原生 SQL 查询
rows, err := db.Raw("SELECT name, age FROM users WHERE age > ?", 18).Rows()
if err != nil {
    panic(err)
}
defer rows.Close()

// 遍历查询结果
for rows.Next() {
    var name string
    var age int
    if err := rows.Scan(&name, &age); err != nil {
        panic(err)
    }
    fmt.Printf("Name: %s, Age: %d\n", name, age)
}
```

### 一般查询

```go
var u User
// 根据主键查询第一条记录
db.First(&user)
//// SELECT * FROM users ORDER BY id LIMIT 1;

// 随机获取一条记录
db.Take(&user)
//// SELECT * FROM users LIMIT 1;

// 根据主键查询最后一条记录
db.Last(&user)
//// SELECT * FROM users ORDER BY id DESC LIMIT 1;

// 查询所有的记录
db.Find(&users)//users为[]User
//// SELECT * FROM users;

// 查询指定的某条记录(仅当主键为整型时可用)
db.First(&user, 10)
//// SELECT * FROM users WHERE id = 10;
```

### Where 条件

```go
// Get first matched record
db.Where("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name = 'jinzhu' limit 1;

// Get all matched records
db.Where("name = ?", "jinzhu").Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu';

// <>
db.Where("name <> ?", "jinzhu").Find(&users)
//// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name in ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
//// SELECT * FROM users WHERE name LIKE '%jin%';

// AND/OR
db.Where("name = ? AND/OR age >= ?", "jinzhu", "22").Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' AND/OR age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
//// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
//// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```

#### Struct & Map查询

```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// 主键的切片
db.Where([]int64{20, 21, 22}).Find(&users)
//// SELECT * FROM users WHERE id IN (20, 21, 22);
```

### Not 条件

```go
db.Not("name", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
//// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;

db.Not([]int64{}).First(&user)
//// SELECT * FROM users ORDER BY id LIMIT 1;

// Plain SQL
db.Not("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE NOT(name = "jinzhu") ORDER BY id LIMIT 1;

// Struct
db.Not(User{Name: "jinzhu"}).First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" ORDER BY id LIMIT 1;
```

### 内联条件

```go
// 根据主键获取记录 (只适用于整形主键)
db.First(&user, 23)
//// SELECT * FROM users WHERE id = 23 LIMIT 1;
// 根据主键获取记录, 如果它是一个非整形主键
db.First(&user, "id = ?", "string_primary_key")
//// SELECT * FROM users WHERE id = 'string_primary_key' LIMIT 1;

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
//// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
//// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
//// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
//// SELECT * FROM users WHERE age = 20;
```

### 额外查询选项

```go
// 为查询 SQL 添加额外的 SQL 操作
db.Set("gorm:query_option", "FOR UPDATE").First(&user, 10)
//// SELECT * FROM users WHERE id = 10 FOR UPDATE;
```

### FirstOrInit

获取匹配的第一条记录，否则根据给定的条件**初始化**一个新的对象 (仅支持 struct 和 map 条件)

```go
// 未找到
db.FirstOrInit(&user, User{Name: "non_existing"})
//// user -> User{Name: "non_existing"}

// 找到
db.Where(User{Name: "Jinzhu"}).FirstOrInit(&user)
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
db.FirstOrInit(&user, map[string]interface{}{"name": "jinzhu"})
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
```

#### Attrs

如果记录未找到，将使用参数初始化 struct.

```go
// 未找到
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = 'non_existing';
//// user -> User{Name: "non_existing", Age: 20}

db.Where(User{Name: "non_existing"}).Attrs("age", 20).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = 'non_existing';
//// user -> User{Name: "non_existing", Age: 20}

// 找到
db.Where(User{Name: "Jinzhu"}).Attrs(User{Age: 30}).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = jinzhu';
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
```

#### Assign

不管记录是否找到，都将参数赋值给 struct.

```go
// 未找到
db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrInit(&user)
//// user -> User{Name: "non_existing", Age: 20}

// 找到
db.Where(User{Name: "Jinzhu"}).Assign(User{Age: 30}).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = jinzhu';
//// user -> User{Id: 111, Name: "Jinzhu", Age: 30}
```

### FirstOrCreate

获取匹配的第一条记录, 否则根据给定的条件**创建**一个新的记录 (仅支持 struct 和 map 条件)

```go
// 未找到
db.FirstOrCreate(&user, User{Name: "non_existing"})
//// INSERT INTO "users" (name) VALUES ("non_existing");
//// user -> User{Id: 112, Name: "non_existing"}

// 找到
db.Where(User{Name: "Jinzhu"}).FirstOrCreate(&user)
//// user -> User{Id: 111, Name: "Jinzhu"}
```

#### Attrs

如果记录未找到，将使用参数创建 struct 和记录.

```go
 // 未找到
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'non_existing';
//// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
//// user -> User{Id: 112, Name: "non_existing", Age: 20}

// 找到
db.Where(User{Name: "jinzhu"}).Attrs(User{Age: 30}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'jinzhu';
//// user -> User{Id: 111, Name: "jinzhu", Age: 20}
```

#### Assign

不管记录是否找到，都将参数赋值给 struct 并保存至数据库.

```go
// 未找到
db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'non_existing';
//// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
//// user -> User{Id: 112, Name: "non_existing", Age: 20}

// 找到
db.Where(User{Name: "jinzhu"}).Assign(User{Age: 30}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'jinzhu';
//// UPDATE users SET age=30 WHERE id = 111;
//// user -> User{Id: 111, Name: "jinzhu", Age: 30}
```

### 高级查询

#### 子查询(推荐使用原生sql语句)

基于 `*gorm.expr` 的子查询

```go
db.Where("amount > ?", db.Table("orders").Select("AVG(amount)").Where("state = ?", "paid").SubQuery()).Find(&orders)
// SELECT * FROM "orders"  WHERE "orders"."deleted_at" IS NULL AND (amount > (SELECT AVG(amount) FROM "orders"  WHERE (state = 'paid')));
```

#### 选择字段

Select，指定你想从数据库中检索出的字段，默认会选择全部字段。

```go
db.Select("name, age").Find(&users)
//// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
//// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
//// SELECT COALESCE(age,'42') FROM users;
```

#### 排序

Order，指定从数据库中检索出记录的顺序。设置第二个参数 reorder 为 `true` ，可以覆盖前面定义的排序条件。

```go
db.Order("age desc, name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// 多字段排序
db.Order("age desc").Order("name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// 覆盖排序
db.Order("age desc").Find(&users1).Order("age", true).Find(&users2)
//// SELECT * FROM users ORDER BY age desc; (users1)
//// SELECT * FROM users ORDER BY age; (users2)
```

#### 数量

Limit，指定从数据库检索出的最大记录数。

```go
db.Limit(3).Find(&users)
//// SELECT * FROM users LIMIT 3;

// -1 取消 Limit 条件
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
//// SELECT * FROM users LIMIT 10; (users1)
//// SELECT * FROM users; (users2)
```

#### 偏移

Offset，指定开始返回记录前要跳过的记录数。

```go
db.Offset(3).Find(&users)
//// SELECT * FROM users OFFSET 3;

// -1 取消 Offset 条件
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
//// SELECT * FROM users OFFSET 10; (users1)
//// SELECT * FROM users; (users2)
```

#### 总数

Count，该 model 能获取的记录总数。

```go
db.Where("name = ?", "jinzhu").Or("name = ?", "jinzhu 2").Find(&users).Count(&count)
//// SELECT * from USERS WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (users)
//// SELECT count(*) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (count)

db.Model(&User{}).Where("name = ?", "jinzhu").Count(&count)
//// SELECT count(*) FROM users WHERE name = 'jinzhu'; (count)

db.Table("deleted_users").Count(&count)
//// SELECT count(*) FROM deleted_users;

db.Table("deleted_users").Select("count(distinct(name))").Count(&count)
//// SELECT count( distinct(name) ) FROM deleted_users; (count)
```

**注意** `Count` 必须是链式查询的最后一个操作 ，因为它会覆盖前面的 `SELECT`，但如果里面使用了 `count` 时不会覆盖

#### Group & Having

```go
rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
for rows.Next() {
  ...
}

// 使用Scan将多条结果扫描进事先准备好的结构体切片中
type Result struct {
	Date time.Time
	Total int
}
var rets []Result
db.Table("users").Select("date(created_at) as date, sum(age) as total").Group("date(created_at)").Scan(&rets)

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```

#### 连接

Joins，指定连接条件

```go
rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
for rows.Next() {
  ...
}

db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)

// 多连接及参数
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)
```

### Pluck

Pluck，查询 model 中的一个列作为切片，如果您想要查询多个列，您应该使用 `Scan`

```go
var ages []int64
db.Find(&users).Pluck("age", &ages)

var names []string
db.Model(&User{}).Pluck("name", &names)

db.Table("deleted_users").Pluck("name", &names)

// 想查询多个字段？ 这样做：
db.Select("name, age").Find(&users)
```

### 扫描

Scan，扫描结果至一个 struct.

```go
type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name, age").Where("name = ?", "Antonio").Scan(&result)

var results []Result
db.Table("users").Select("name, age").Where("id > ?", 0).Scan(&results)

// 原生 SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```

## 链式操作相关

```go
// 创建一个查询
tx := db.Where("name = ?", "jinzhu")

// 添加更多条件
if someCondition {
  tx = tx.Where("age = ?", 20)
} else {
  tx = tx.Where("age = ?", 30)
}

if yetAnotherCondition {
  tx = tx.Where("active = ?", 1)
}
```

基于scopes，你可以抽取一些通用逻辑，写出更多可重用的函数库。

```go
func AmountGreaterThan1000(db *gorm.DB) *gorm.DB {
  return db.Where("amount > ?", 1000)
}

func PaidWithCreditCard(db *gorm.DB) *gorm.DB {
  return db.Where("pay_mode_sign = ?", "CR")
}

func PaidWithCod(db *gorm.DB) *gorm.DB {
  return db.Where("pay_mode_sign = ?", "COD")
}

func OrderStatus(status []string) func (db *gorm.DB) *gorm.DB {
  return func (db *gorm.DB) *gorm.DB {
    return db.Scopes(AmountGreaterThan1000).Where("status IN (?)", status)
  }
}

db.Scopes(AmountGreaterThan1000, PaidWithCreditCard).Find(&orders)
// 查找所有金额大于 1000 的信用卡订单

db.Scopes(AmountGreaterThan1000, PaidWithCod).Find(&orders)
// 查找所有金额大于 1000 的 COD 订单

db.Scopes(AmountGreaterThan1000, OrderStatus([]string{"paid", "shipped"})).Find(&orders)
// 查找所有金额大于 1000 且已付款或者已发货的订单
```

## 更新

> 先查询后修改 或 使用空结构体，条件写在Where中

#### 保存所有字段

`Save` 会保存所有的字段，即使字段是零值

```go
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id = 1;
```

### 更新修改字段

```go
// 更新单个属性，如果它有变化
db.Model(&user).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据给定的条件更新单个属性
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;

// 使用 map 更新多个属性，只会更新其中有变化的属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// 使用 struct 更新多个属性，只会更新其中有变化且为非零值的字段
db.Model(&user).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 警告：当使用 struct 更新时，GORM只会更新那些非零值的字段
// 对于下面的操作，不会发生任何更新，"", 0, false 都是其类型的零值
db.Model(&user).Updates(User{Name: "", Age: 0, Active: false})
```

### 更新选定字段

如果你想更新或忽略某些字段，你可以使用 `Select`，`Omit`

```go
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

### 无Hooks更新

上面的更新操作会自动运行 model 的 `BeforeUpdate`, `AfterUpdate` 方法，更新 `UpdatedAt` 时间戳, 在更新时保存其 `Associations`, 如果你不想调用这些方法，你可以使用 `UpdateColumn`， `UpdateColumns`

```go
// 更新单个属性，类似于 `Update`
db.Model(&user).UpdateColumn("name", "hello")
//// UPDATE users SET name='hello' WHERE id = 111;

// 更新多个属性，类似于 `Updates`
db.Model(&user).UpdateColumns(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18 WHERE id = 111;
```

### 批量更新

批量更新时`Hooks（钩子函数）`不会运行。

```go
db.Table("users").Where("id IN (?)", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
//// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);

// 使用 struct 更新时，只会更新非零值字段，若想更新所有字段，请使用map[string]interface{}
db.Model(User{}).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18;

// 使用 `RowsAffected` 获取更新记录总数
db.Model(User{}).Updates(User{Name: "hello", Age: 18}).RowsAffected
```

### 使用SQL表达式更新

先查询表中的第一条数据保存至user变量。

```go
var user User
db.First(&user)
db.Model(&user).Update("age", gorm.Expr("age * ? + ?", 2, 100))
//// UPDATE `users` SET `age` = age * 2 + 100, `updated_at` = '2020-02-16 13:10:20'  WHERE `users`.`id` = 1;

db.Model(&user).Updates(map[string]interface{}{"age": gorm.Expr("age * ? + ?", 2, 100)})
//// UPDATE "users" SET "age" = age * '2' + '100', "updated_at" = '2020-02-16 13:05:51' WHERE `users`.`id` = 1;

db.Model(&user).UpdateColumn("age", gorm.Expr("age - ?", 1))
//// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1';

db.Model(&user).Where("age > 10").UpdateColumn("age", gorm.Expr("age - ?", 1))
//// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1' AND quantity > 10;
```

### 修改Hooks中的值

如果你想修改 `BeforeUpdate`, `BeforeSave` 等 Hooks 中更新的值，你可以使用 `scope.SetColumn`, 例如：

```go
func (user *User) BeforeSave(scope *gorm.Scope) (err error) {
  if pw, err := bcrypt.GenerateFromPassword(user.Password, 0); err == nil {
    scope.SetColumn("EncryptedPassword", pw)
  }
}
```

### 其它更新选项

```go
// 为 update SQL 添加其它的 SQL
db.Model(&user).Set("gorm:update_option", "OPTION (OPTIMIZE FOR UNKNOWN)").Update("name", "hello")
//// UPDATE users SET name='hello', updated_at = '2013-11-17 21:34:10' WHERE id=111 OPTION (OPTIMIZE FOR UNKNOWN);
```

## 删除

### 删除记录

**警告** 删除记录时，请确保主键字段有值，GORM 会通过主键去删除记录，如果主键为空，GORM 会删除该 model 的所有记录。

```go
// 删除现有记录
db.Delete(&email)
//// DELETE from emails where id=10;

// 为删除 SQL 添加额外的 SQL 操作
db.Set("gorm:delete_option", "OPTION (OPTIMIZE FOR UNKNOWN)").Delete(&email)
//// DELETE from emails where id=10 OPTION (OPTIMIZE FOR UNKNOWN);
```

### 批量删除

删除全部匹配的记录

```go
db.Where("email LIKE ?", "%jinzhu%").Delete(Email{})
//// DELETE from emails where email LIKE "%jinzhu%";

db.Delete(Email{}, "email LIKE ?", "%jinzhu%")
//// DELETE from emails where email LIKE "%jinzhu%";
```

### 软删除

如果一个 model 有 `DeletedAt` 字段，他将自动获得软删除的功能！ 当调用 `Delete` 方法时， 记录不会真正的从数据库中被删除， 只会将`DeletedAt` 字段的值会被设置为当前时间

```go
db.Delete(&user)
//// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// 批量删除
db.Where("age = ?", 20).Delete(&User{})
//// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// 查询记录时会忽略被软删除的记录
db.Where("age = 20").Find(&user)
//// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;

// Unscoped 方法可以查询被软删除的记录
db.Unscoped().Where("age = 20").Find(&users)
//// SELECT * FROM users WHERE age = 20;
```

### 物理删除

```go
// Unscoped 方法可以物理删除记录
db.Unscoped().Delete(&order)
//// DELETE FROM orders WHERE id=10;
```
