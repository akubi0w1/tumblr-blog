date: 2020/08/04

## golangのscanでぬるぽを避ける

## is なに

`row.Scan()`したときに、DB側で`null`の値があると、読めへんわって言われる。これをどう避けるか。

```
sql: Scan error on column index 2, name "xxx": unsupported Scan, storing driver.Value type <nil> into type *string
```

## 解決策

`bio`というカラムがNULL許可。避けてみる。

### 1. database/sqlパッケージのNULL許可の型を使う

```go
row := DB.QueryRow("SELECT id, name, bio FROM users WHERE id=2")
var id int
var name string
var _bio sql.NullString // null許容の型

if err := row.Scan(&id, &name, &_bio); err != nil {
	log.Println("scan error: ", err)
	w.Write([]byte("Scan error"))
	return
}

// nullチェック
var bio string
if _bio.Valid {
	// _bio.Stringで値の取得
	bio = _bio.String
} else {
	// _bioがnull
	bio = "bio is null"
}
```

毎回バリデーションすんのかあ...という気持ちになりました。

### 2. SQLのクエリで対応する

SQLの`COALESCE()`を使って対応。コーアレスって読むらしい。
`COALESCE(column1, column2, ...)`、column1 -> column2 -> ...って見ていき、一番最初にNULLでなかった値を取得するらしい。

つまり、`COALESCE(column1, value)`とかってしておけば、nullだったときに代替の値(`value`)を突っ込める。

```go
row := DB.QueryRow(`
	SELECT id, name, COALESCE(bio, 'bio is null')
	FROM users
	WHERE id=2`)

var id int
var name string
var bio string
if err := row.Scan(&id, &name, &bio); err != nil {
	log.Println("scan error: ", err)
	w.Write([]byte("Scan error"))
	return
}
str := fmt.Sprintf("id: %d, name: %s, bio: %s", id, name, bio)
w.Write([]byte(str))
```

後者の方が好みだった...。NULL許可との戦いでした。

## 参考

- [https://golang.shop/post/go-databasesql-08-nulls-ja/](https://golang.shop/post/go-databasesql-08-nulls-ja/)
- [https://qiita.com/mikakane/items/1e45c2a798d0c7edffda](https://qiita.com/mikakane/items/1e45c2a798d0c7edffda)
