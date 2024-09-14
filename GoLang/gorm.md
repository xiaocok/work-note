

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
	// joinForeignKey: 用于定义多对多关系中的外键字段。
	// joinReferences: 用于指定多对多关系中被引用的主键字段。
	// joinForeignKey: 指定中间表中的字段，该字段作为外键引用另一个表。
	// joinReferences: 指定被引用的表中的字段。
    // many2many：指定多对多的关系表
	Groups []Group `gorm:"many2many:user_group;joinForeignKey:user_id;joinReferences:id"`
}

type UserGroup struct {
	ID        uint    `gorm:"column:id"`
	UserID    uint    `gorm:"column:user_id"`
	GroupID   uint    `gorm:"column:group_id"`
}

type Group struct {
	ID   uint   `gorm:"column(id)"`
	Name string `gorm:"column(name)"`
	// 如果不需要查询User，这里不需要指定
    // 如果需要查询group的user，这里需要定义user和gorm的tag。如果只查询user的group
	Users []User `gorm:"many2many:user_group;joinForeignKey:group_id;joinReferences:group_id"`
}
```

```go
var users []User
gormDB.Preload("Groups").Find(&users) // 获取用户及其关联的群组
```

