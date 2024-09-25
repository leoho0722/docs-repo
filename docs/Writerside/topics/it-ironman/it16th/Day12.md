# 【Go！帶你探索 FIDO2 資安技術全端應用】Day 12 - 連接資料庫跟建立資料表

昨天定義好 Request 和 Response body，今天要先來連接資料庫與建立資料表

## 連接資料庫跟建立資料表

### 安裝 PostgreSQL 資料庫 {id="postgresql_1"}

為了將註冊好的使用者儲存起來，所以需要將資料持久性儲存在資料庫中

這邊使用 PostgreSQL 作為我們的資料庫選擇，並使用 docker compose 來建立服務

我們先新增一個 docker 資料夾來存放 `docker-compose.yml`

![新增 docker 資料夾](https://ithelp.ithome.com.tw/upload/images/20240913/20140363AFLIM8Npgb.png)
▲ 新增 docker 資料夾

並在資料夾內新增一個檔案叫做 `docker-compose.yml`，貼上下面的 code 即可透過 docker compose 安裝好 PostgreSQL 資料庫

```yaml
services:
  postgres:
    image: postgres:16.4
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: it16th
    volumes:
      - ../postgres:/var/lib/postgresql/data
```

要將服務 run 起來的話，只要透過一行指令 `docker compose up` 就可以快速將服務 run 起來了

### 連線到 PostgreSQL

接著，我們在程式中要跟資料庫進行連線，這邊是使用 [GORM](https://gorm.io) 作為 ORM (Object Relational Mapping)
安裝方法也很簡單，一樣就一行指令就可以安裝好了

```shell
go get -u gorm.io/gorm
```

透過上面的指令安裝好 GORM 後，接著要再安裝 PostgreSQL 的 GORM driver，一樣透過下面指令就可以安裝好了

```shell
go get -u gorm.io/driver/postgres
```

將這兩個都安裝好之後，接著新增一個 database 的資料夾，並新增兩個檔案，分別為 `connect.go`、`model.go`，package 都是 database

#### connect.go

這邊我們只要提供資料庫的連線參數 (dsn)，透過 GORM 提供的 `gorm.Open` 和 `postgres.Open` 就可以連接到前面 run 起來的 PostgreSQL 資料庫了

接著就可以向資料庫建立出我們想要的資料表，在這邊是 `User` 資料表

```go
package database

import (
	"fmt"
	"sync"

	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

type context struct {
	mu sync.Mutex
	db *gorm.DB
}

var Context *context

func Connect() {
	dsn := fmt.Sprintf(
		"host=%s port=%s user=%s password=%s dbname=%s sslmode=disable",
		"localhost", "5432", "postgres", "postgres", "it16th",
	)
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		panic("failed to connect database")
	}

	db.AutoMigrate(&User{})

	migrator := db.Migrator()
	migrator.HasTable(&User{})

	Context = &context{db: db}
}
```

#### model.go

我們會需要儲存使用者資料，那麼就會需要有對應的資料表，下面就來建立對應的資料表

在資料表中，主要會包含五個欄位

1. ID
    * 說明：使用者 ID，會使用 UUID 進行生成
2. Name
    * 說明：使用者名稱
3. DisplayName
    * 說明：使用者的顯示名稱
4. Challenge
    * 說明：當次進行 WebAuthn 註冊 / 驗證流程時的使用者 Challenge
5. Credential
    * 說明：使用者的 WebAuthn Credential

最下面的 method `func (*User) TableName() string` 是用來設定這張資料表的名稱

```go
package database

type User struct {
	// ID 使用者 ID
	ID string `json:"userId" gorm:"primaryKey"`

	// Name 使用者名稱
	Name string `json:"name"`

	// DisplayName 使用者的顯示名稱
	DisplayName string `json:"displayName"`

	// Challenge 當次進行 WebAuthn 註冊 / 驗證流程時的使用者 Challenge
	Challenge string `json:"challenge"`

	// Credential 使用者的 WebAuthn Credential
	Credential string `json:"credential"`
}

func (*User) TableName() string {
	return "user"
}
```

### 在 main.go 中呼叫

撰寫好上述的 Function 後，接著到 main.go 呼叫

```go
package main

import (
	"leoho.io/it16th-webauthn-rp-server/database"
	"leoho.io/it16th-webauthn-rp-server/route"
)

func main() {
	database.Connect()
	route.NewRoute()
}
```

今天我們將用來儲存使用者資料的資料庫建立好之後，後面在設計 Controller 時，就可以處理 WebAuthn 所需的資料庫操作了～