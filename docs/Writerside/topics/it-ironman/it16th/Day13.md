# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 13 - 撰寫 Repository 功能

昨天連線到資料庫後，今天要來撰寫負責與資料庫互動的 Repository 功能

一共有四個功能，也就是 CRUD (Create, Read, Update, Delete)

透過使用 GORM 可以讓我們快速設計出 CRUD 功能

## 設計 CRUD 功能

在 database 資料夾中新增一個檔案，叫做 `repository.go`，package 一樣是 database

### Create

首先是 Create，透過 GORM 提供的 `Create` method 可以直接新增資料到資料庫中
這邊使用了 mutex 來確保線程安全 (Thread-safe)

```go
// CreateUser 建立使用者
func CreateUser(u *User) error {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	return Context.db.Create(u).Error
}
```

對應的 SQL 語句如下

```sql
INSERT INTO "user" (id, display_name, challenge, credential)
VALUES ('user123', 'John Doe', 'example_challenge', 'example_credential');
```

### Read

接著是 Read，這邊會設計四種從資料庫中讀取的方式，分別為

1. 透過使用者 ID 取得使用者 (`GetUserByID`)
2. 透過使用者 Name 取得使用者 (`GetUserByName`)
3. 透過使用者 Challenge 取得使用者 (`GetUserByChallenge`)
4. 取得所有使用者 (`GetUsers`)

#### 透過使用者 ID 取得使用者

```go
// GetUserByID 透過 UserID 取得使用者
func GetUserByID(id string) (*User, error) {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	var u User
	err := Context.db.Where("id = ?", id).First(&u).Error
	return &u, err
}
```

對應的 SQL 語句如下

```sql
SELECT * FROM "user" WHERE id = 'user123' LIMIT 1;
```

#### 透過使用者 Name 取得使用者

```go
// GetUserByName 透過 Name 取得使用者
func GetUserByName(name string) (*User, error) {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	var u User
	err := Context.db.Where("name = ?", name).First(&u).Error
	return &u, err
}
```

對應的 SQL 語句如下

```sql
SELECT * FROM "user" WHERE name = 'user123' LIMIT 1;
```

#### 透過使用者 Challenge 取得使用者

```go
// GetUserByChallenge 透過 Challenge 取得使用者
func GetUserByChallenge(challenge string) (*User, error) {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	var u User
	err := Context.db.Where("challenge = ?", challenge).First(&u).Error
	return &u, err
}
```

對應的 SQL 語句如下

```sql
SELECT * FROM User WHERE challenge = '5NZzKdhQ7LDdJLofZNyuz7GTYws27mrPXcsktG9PuB0' LIMIT 1;
```

#### 取得所有使用者

```go
// GetUsers 取得所有使用者
func GetUsers() ([]User, error) {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	var users []User
	err := Context.db.Find(&users).Error
	return users, err
}
```

對應的 SQL 語句如下

```sql
SELECT * FROM "user";
```

### Update

再來是 Update，透過 updateData 來更新對應的使用者，這邊 updateData 是使用 `interface{}` 作為型別，因為可能在傳入時，是使用 `map` 或是 `User` 進行更新
一樣使用 mutex 來確保線程安全

```go
// UpdateUser 更新使用者
func UpdateUser(u *User, updateData interface{}) error {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	return Context.db.Model(u).Updates(updateData).Error
}
```

對應的 SQL 語句如下

```sql
UPDATE User SET challenge = 'cnikcnononeoc' WHERE id = 'user123';
```

### Delete

最後是 Delete，透過使用 id 來刪除對應使用者
一樣使用 mutex 來確保線程安全

```go
// DeleteUserByID 透過 UserID 刪除使用者
func DeleteUserByID(id string) error {
	Context.mu.Lock()
	defer Context.mu.Unlock()

	return Context.db.Delete(&User{ID: id}).Error
}
```

對應的 SQL 語句如下

```sql
DELETE FROM User WHERE id = 'user123';
```

撰寫好存取資料庫的 CRUD 功能後，明天就可以開始來進行 WebAuthn 相關操作了～