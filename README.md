# Golang SQL Control

This module provides a simple interface for interacting with SQL databases. Testing was conducted with MySQL and SQLite3

The module implements the following functionality:
- Tables:
    - Creation
    - Migration (automatic or manual)
    - Deletion
- Values:
    - Select
    - Insert
    - Update
    - Delete

## Installation
```
go get github.com/GlshchnkLx/go_mod_sqlctrl
```

```golang
import (
	sqlctrl "github.com/GlshchnkLx/go_mod_sqlctrl"
)
```

## Examples

```golang
package main

import (
	"fmt"

	sqlctrl "github.com/GlshchnkLx/go_mod_sqlctrl"
	_ "github.com/go-sql-driver/mysql"
	_ "modernc.org/sqlite"
)

// tagging structures for the module
type User struct {
	ID   int64  `sql:"id, INTEGER, PRIMARY_KEY, AUTO_INCREMENT"`
	Name string `sql:"name, TEXT(32)"`
}

type Token struct {
	ID      int64  `sql:"id, INTEGER, PRIMARY_KEY, AUTO_INCREMENT"`
	Content string `sql:"content, TEXT(64), UNIQUE"`
}

type UserToken struct {
	ID int64 `sql:"id, INTEGER, PRIMARY_KEY, AUTO_INCREMENT"`

	UserID  int64 `sql:"user_id"`
	TokenID int64 `sql:"token_id"`
}

type UserTokenView struct {
	UserID       int64  `sql:"user_id"`
	TokenID      int64  `sql:"token_id"`
	UserName     string `sql:"user_name"`
	TokenContent string `sql:"token_content"`
}

func main() {
	// connect the database and specify the schema file
	db, err := sqlctrl.NewDatabase("sqlite", "./example.db", "./example.json")
	_ = err

	// creating a table with the name
	userTable, err := sqlctrl.NewTable("users", User{})

	// starting the automatic migration procedure
	err = db.MigrationTable(userTable, nil)

	tokenTable, err := sqlctrl.NewTable("tokens", Token{})

	err = db.MigrationTable(tokenTable, nil)

	userTokenTable, err := sqlctrl.NewTable("user_token", UserToken{})

	err = db.MigrationTable(userTokenTable, nil)

	// creating a view without the name
	userTokenViewTable, err := sqlctrl.NewTable("", UserTokenView{})

	// inserting a single records into the table
	_, err = db.InsertValue(userTable, User{
		Name: "User 1",
	})

	_, err = db.InsertValue(userTable, User{
		Name: "User 2",
	})

	// inserting an array of records into a table
	_, err = db.InsertValue(tokenTable, []Token{
		{
			Content: "Token 1",
		}, {
			Content: "Token 2",
		}, {
			Content: "Token 3",
		},
	})

	_, err = db.InsertValue(userTokenTable, []UserToken{
		{
			UserID:  1,
			TokenID: 1,
		}, {
			UserID:  2,
			TokenID: 2,
		}, {
			UserID:  1,
			TokenID: 3,
		},
	})

	// making a request for view
	resultInterface, err := db.Query(userTokenViewTable, `
		SELECT
				users.id AS user_id,
				tokens.id AS token_id,
				users.name AS user_name,
				tokens.content AS token_content
		FROM users, tokens, user_token
		WHERE users.id = user_token.user_id AND tokens.id = user_token.token_id
	`)

	// output result
	resultArray := resultInterface.([]UserTokenView)
	for i, v := range resultArray {
		fmt.Println(i, v.UserName, v.TokenContent)
	}
}
```

## Testing
```
go test -v
```