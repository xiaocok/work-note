

### 唯一键



### 关联查询

#### 一对一

```go
type User struct {
	ID          uint   `gorm:"column(id)"`
	Name        string `gorm:"column(name)"`
	// foreignKey是另外一张表的字段，references是本表的字段
	// 关联关系存另外的表
    Profile     *Profile `gorm:"foreignKey:user_id;references:id"` // 与 Profile 表建立一对一的关系
    
    // 关联关系存在本表
    ProfileName string `gorm:"column(profile_name)"`
    Profile     *Profile `gorm:"foreignKey:name;references:profile_name"` // 与 Profile 表建立一对一的关系
}

type Profile struct {
	ID      uint
	Name    string
	UserID  uint
	Email   string
	Address string
}
```

```go
var users []User
// Preload使用的是结构体中定义的字段，而不是数据库列名称
gormDB.Preload("Profile").Find(&users)
```

#### 一对多

```go
type User struct {
	ID          uint   `gorm:"column(id)"`
	Name        string `gorm:"column(name)"`
	ProfileName string `gorm:"column(profile_name)"`
	// foreignKey是另外一张表的字段，references是本表的字段
	Posts []Post `gorm:"foreignKey:user_id;references:id"` // 一个用户可以有多个帖子
}

type Post struct {
	ID       uint      `gorm:"column(id)"`
	Title    string    `gorm:"column(title)"`
	UserID   uint      `gorm:"column(user_id)"`
	Comments []Comment `gorm:"foreignKey:post_id;references:id"` // 一个帖子有多个评论
}

type Comment struct {
	ID      uint   `gorm:"column(id)"`
	Content string `gorm:"column(content)"`
	PostID  uint   `gorm:"column(post_id)"`
}
```

```go
var users []User
// Preload使用的是结构体中定义的字段，而不是数据库列名称
// Posts是User结构体定义的字段，Comments是Post结构体定义的字段
// 支持多级关联查询
gormDB.Preload("Posts.Comments").Find(&users)
```

#### 多对多

> 多对多需要中间表

```go
type User struct {
	ID          uint   `gorm:"column(id)"`
	Name        string `gorm:"column(name)"`
    // foreignKey: 这个字段用来指定在关联表中用于表示本表记录的外键字段名。不指定，默认是本表的id字段：user.id。
    // references: 指定当前表的字段引用的另一个表的字段名。不指定则为另外一个表的id字段：group.id。
    // joinForeignKey: 在多对多关联的中间表中，用来标识当前模型记录的字段：user_group表。
	// joinReferences: 在多对多关联的中间表中，用来标识另一个模型记录的字段：user_group表。
    // many2many：指定多对多的关系表：user_group
	Groups []Group `gorm:"many2many:user_group;joinForeignKey:user_id;joinReferences:group_id"`
}

type UserGroup struct {
	ID        uint    `gorm:"column:id"`
	UserID    uint    `gorm:"column:user_id"`
	GroupID   uint    `gorm:"column:group_id"`
}

type Group struct {
	ID   uint   `gorm:"column(id)"`
	Name string `gorm:"column(name)"`
}
```

```go
var users []User
gormDB.Preload("Groups").Find(&users) // 获取用户及其关联的群组
```

> 自定义关联关系

```go
type User struct {
	ID          uint   `gorm:"column(id)"`
	Name        string `gorm:"column(name)"`
    // foreignKey: 这个字段用来指定在关联表中用于表示本表记录的外键字段名：user.name。
    // references: 指定当前表的字段引用的另一个表的字段名：group.name。
    // joinForeignKey: 在多对多关联的中间表中，用来标识当前模型记录的字段：user_group.user_name。
    // joinReferences: 在多对多关联的中间表中，用来标识另一个模型记录的字段：user_group.group_name。
    // many2many：指定多对多的关系表：user_group
	Groups []Group `gorm:"many2many:user_group;foreignKey:name;joinForeignKey:user_name;references:name;joinReferences:group_name"`
}

type UserGroup struct {
	ID uint `gorm:"column:id"`
	UserName  string  `gorm:"column:user_name"`
	GroupName string  `gorm:"column:group_name"`
}

type Group struct {
	ID   uint   `gorm:"column(id)"`
	Name string `gorm:"column(name)"`
}
```

```go
var users []User
// 定义各自的关联关系，使用多级关联模式查询
gormDB.Preload("Groups").Find(&users) // 获取用户及其关联的群组
```

#### 多字段关联

```go
type User struct {
	ID          uint   `gorm:"column(id)"`
	Name        string `gorm:"column(name)"`
	Type        string `gorm:"column(type)"`
	// foreignKey是另外一张表的字段，references是本表的字段
	Profile   *Profile `gorm:"foreignKey:user_name,user_type;references:name,type"` // 多个字段使用都好隔开
}

type Profile struct {
	Id       uint   `gorm:"column:id"`
	UserName string `gorm:"column:user_name"`
	UserType string `gorm:"column:user_type"`
}
```

```go
var users []User
gormDB.Preload("Profile").Find(&users)
```

#### Join

```go
type User struct {
	ID   uint   `gorm:"column:id"`
	Name string `gorm:"column(name)"`
	Type string `gorm:"column(type)"`
}

type Profile struct {
	ID       uint   `gorm:"column:id"`
	UserName string `gorm:"column:user_name"`
	UserType string `gorm:"column:user_type"`
}

type UserProfile struct{
	User
	Profile
}
```

```go
var userProfiles []UserProfile

gormDB.Table("user").           // gormDB.Table("user") 或者 gormDB.Table(new(User))都可以
    Select("user.*, profile.*").
    Joins("LEFT JOIN profile ON profile.user_name = user.name AND profile.user_type = user.type").
    Where("profile.id = ?", 1). // 可以添加额外的查询条件
    Scan(&userProfiles)         // Scan方法用于填充userProfiles变量，它应该是一个包含Profile和User字段的结构体
```

#### Scopes

```go
type User struct {
	ID   uint   `gorm:"column:id"`
	Name string `gorm:"column(name)"`
	Type string `gorm:"column(type)"`
}

type Profile struct {
	ID       uint   `gorm:"column:id"`
	UserName string `gorm:"column:user_name"`
	UserType string `gorm:"column:user_type"`
}

type UserProfile struct{
	User
	Profile
}
```

```go
var userProfiles []UserProfile

gormDB.Model(new(User)).Select("user.*,profile.*").Scopes(
    func(db *gorm.DB) *gorm.DB {
        return db.Joins("LEFT JOIN profile ON profile.name = user.name AND profile.email = user.email")
    },
    func(db *gorm.DB) *gorm.DB {
        return db.Where("user.id in ?", []int{11, 12})
    },
).Scan(&userProfiles)
```

