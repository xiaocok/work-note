

### 唯一键



### 关联查询

#### 一对一

> foreignKey：含有外键的那张表的外键对应的数据库字段或者结构体字段【外键字段】
>
> references：没有外键的那张表，与外键形成关联关系的引用的数据库字段或者结构体字段【与外键关联关系的字段】

默认值

> foreignKey：没有外键那张表名称+ID【主表/非外键表名称+ID字段】
>
> references：没有外键那张表的ID字段【主表/非外键表的ID字段】



##### 外键在本表里面：BelongsTo

```go
// 表名默认为复数users
type User struct {
	ID   uint   `gorm:"column(id)"`
	Name string `gorm:"column(name)"`
    
	// foreignKey：User表CompanyID[company_id]字段作为company表的外键，即foreignKey为CompanyID[company_id]
	// references: 建立关联关系的引用字段：company表的ID[id]字段
	CompanyID       int      `gorm:"column(company_id)"`    // 默认：foreignKey=非外键表的表名称+ID，即User.CompanyID。references=非外键表的ID字段，即Company.ID
	Company         *Company // 外键字段名称，默认为结构体名称+ID。这里是CompanyID[company_id]作为foreignKey关联字段查询
	CompanyByFiled  *Company `gorm:"foreignKey:CompanyID"`  // 指定User存储的Company表的外键的结构体字段名称[CompanyID]
	CompanyByColumn *Company `gorm:"foreignKey:company_id"` // 指定User存储的Company表的外键的数据库字段名称[company_id]

	// 如果不是使用的ID作为外键，需要指定自定义外键名字+另外一个表需要建立关联关系的引用字段的名字
	CompanyName         string   `gorm:"column(company_name)"`
	CompanyByNameFiled  *Company `gorm:"foreignKey:CompanyName;references:Name"`  // 指定User存储的Company表的外键的结构体字段名称[CompanyName]，引用Company结构体的字段Name
	CompanyByNameColumn *Company `gorm:"foreignKey:company_name;references:name"` // 指定User存储的Company表的外键的数据库字段名称[company_name]，引用Company表的字段name
    
    // 多字段关联
	CompanyMultiFiled  *Company `gorm:"foreignKey:CompanyID,CompanyName;references:ID,Name"`   // 多字端关联：通过User结构体字段名关联
	CompanyMultiColumn *Company `gorm:"foreignKey:company_id,company_name;references:id,name"` // 多字端关联：通过User表列名关联
}

// 表名默认为复数companies：注意特殊的复数表明
type Company struct {
	ID   int    `gorm:"column(id)"`
	Name string `gorm:"column(name)"`
}
```

```go
var users []User
gormDB.Preload("Company").Find(&users)             // 默认方式
gormDB.Preload("CompanyByFiled").Find(&users)      // 通过User结构体字段名关联
gormDB.Preload("CompanyByColumn").Find(&users)     // 通过User表列名关联
gormDB.Preload("CompanyByNameFiled").Find(&users)  // 关联字段非ID：通过User结构体字段名关联
gormDB.Preload("CompanyByNameColumn").Find(&users) // 关联字段非ID：通过User表列名关联
gormDB.Preload("CompanyMultiFiled").Find(&users)   // 多字段关联：通过User结构体字段名关联
gormDB.Preload("CompanyMultiColumn").Find(&users)  // 多字段关联：通过User表列名关联
```



##### 外键在另外一张表：Has One

> foreignKey：User表ProfileID[profile_id]字段作为Profile表的外键，即foreignKey为ProfileID
>
> references: 另外一个表需要建立关联关系的引用字段的名字

```go
// 表名默认为复数users
type User struct {
	ID   uint   `gorm:"column(id)"`
	Name string `gorm:"column(name)"`

	Profile *Profile // 默认：foreignKey=非外键表的表名称+ID，即Profile.UserID。references=非外键表的ID字段，即User.ID
	// foreignKey：profiles表UserID[user_id]字段作为users.id的外键，即foreignKey为UserID[user_id]
	// references: 建立关联关系的引用字段的名字：User.ID[user.id]
	ProfileByFiled  *Profile `gorm:"foreignKey:UserID"` // 默认使用users.id作为引用字段references
	ProfileByColumn *Profile `gorm:"foreignKey:user_id"`

	// 自定义外键和引用字段
	// foreignKey：作为users.Name[users.name]字段外键的profiles.UserName[profiles.user_name]
	// references：建立关联关系的引用字段的名字users.Name[users.name]
	ProfileByNameFiled  *Profile `gorm:"foreignKey:UserName;references:Name"`  // 指定User存储的Profile表的外键的结构体字段名称
	ProfileByNameColumn *Profile `gorm:"foreignKey:user_name;references:name"` // 指定User存储的Profile表的外键的数据库字段名称

	// 多字段关联
	// foreignKey：作为外键的profiles表的UserID和UserName
	// references：建立关联关系的引用字段的名字users表的ID和Name
	ProfileMultiFiled  *Profile `gorm:"foreignKey:UserID,UserName;references:ID,Name"`   // 多字端关联：通过User结构体字段名关联
	ProfileMultiColumn *Profile `gorm:"foreignKey:user_id,user_name;references:id,name"` // 多字端关联：通过User表列名关联
}
type Profile struct {
	Id       uint   `gorm:"column(id)"`
	Name     string `gorm:"column(name)"`
	UserID   uint   `gorm:"column(user_id)"`
	UserName string `gorm:"column(user_name)"`
	Email    string `gorm:"column(email)"`
	Address  string `gorm:"column(address)"`
}
```

```go
var users []User
gormDB.Preload("Profile").Find(&users)             // 默认方式
gormDB.Preload("ProfileByFiled").Find(&users)      // 重写外键：通过User结构体字段名关联
gormDB.Preload("ProfileByColumn").Find(&users)     // 重写外键：通过User表列名关联
gormDB.Preload("ProfileByNameFiled").Find(&users)  // 重写引用：关联字段非ID：通过User结构体字段名关联
gormDB.Preload("ProfileByNameColumn").Find(&users) // 重写引用：关联字段非ID：通过User表列名关联
gormDB.Preload("ProfileMultiFiled").Find(&users)   // 多字段关联：通过User结构体字段名关联
gormDB.Preload("ProfileMultiColumn").Find(&users)  // 多字段关联：通过User表列名关联
```

#### 一对多：Has Many

```go
type User struct {
	ID          uint   `gorm:"column(id)"`
	Name        string `gorm:"column(name)"`
	ProfileName string `gorm:"column(profile_name)"`
    // foreignKey：外键的字段
    // references：与外键关联关系的字段
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

例子2

```go
// foreignKey：指定外键
// joinForeignKey：指定连接表的外键
// References：指定引用
// joinReferences：指定连接表的引用外键

type User struct {
    gorm.Model
    Profiles []Profile `gorm:"many2many:user_profiles;foreignKey:Refer;joinForeignKey:UserReferID;References:UserRefer;joinReferences:ProfileRefer"`
    Refer    uint      `gorm:"index:,unique"`
}

type Profile struct {
    gorm.Model
    Name      string
    UserRefer uint `gorm:"index:,unique"`
}

type Profile struct {
    UserReferID   uint `gorm:"column(user_refer_id)"`
    ProfileRefer uint `gorm:"column(profile_refer)"`
}
// 连接表：user_profiles
//   foreign key: user_refer_id, reference: users.refer
//   foreign key: profile_refer, reference: profiles.user_refer
```

```go
var users []User
gormDB.Preload("Profiles").Find(&users)
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

