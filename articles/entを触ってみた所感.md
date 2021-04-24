date: 2021/04/18

# entを触ってみた所感

## ent is なに？

- データモデルをグラフ構造で扱える
- スキーマの定義をGoのコードで行える
- コード生成をベースとした静的型つけ

など...の特徴を持ったエンティティフレームワーク。まあつまりORM。
MySQL、PostgreSQL、SQLite、Gremlinと、よく使うドライバは現状ですでにサポートされている。

## 早速触ってみる

公式の[Quick Introduction](https://entgo.io/docs/getting-started/)を見つつ、いろいろやってみます。
今回やったコードは[github](https://github.com/akubi0w1/ent-sample)に置いてあります。
実際にデータとったり、作成したりはあまりやりません。スキーマの作成あたりにスポットを当てます。

今回は、以下なDBができるようにしてみます。

![](https://64.media.tumblr.com/0a15c92f6da45e228963a4a417b4b9f6/77c86d5077b5095a-6d/s540x810/81409e3ee4d0d2f7bb057ef4f0e3ef7937c5f208.png)

環境は以下を使います。

- go 1.16
- mysql 5.7

### install

```shell
$ go get entgo.io/ent/cmd/ent
```

`ent`をコード生成ツールとして使うので、PATHが通ってる必要があります。
もしとおってないなら、`go run entgo.io/ent/cmd/ent <command>`でも動かせるっぽい。

### create scheme

プロジェクトルートのディレクトリにて、`$ ent init`を使ってスキーマの原型を作成します。

```shell
# 単発で作る
$ ent init User

# 複数を一度に作ることも可能
$ ent init Profile Post Tag
```

そうすると、ルートディレクトリに`ent`ディレクトリが生まれます。中身は以下な感じ。

```plantext
ent
├── generate.go
└── schema
    ├── post.go
    ├── profile.go
    ├── tag.go
    └── user.go
```

`shcema/`にスキーマの原型が入っています。`generate.go`はスキーマを元にしたモデルやメソッドを生成するためのgeneratorがコメントで書かれています。

ちなみに、`ent`ディレクトリですが、出力先を変えることもできます。

```
# --targetを指定してshcemaの出力先を変更する
$ ent init --target mysql/ent/schema <schemas...>
```

ただし、これをやると、`generate.go`は生成されない模様...。なので、自作しちゃいましょう。コメント行さえあっていればおけ。

```go
package 任意のパッケージ
//go:generate go run -mod=mod entgo.io/ent/cmd/ent generate ./schema
```

また、作成されるスキーマのパッケージ名は固定で`shcema`なので、targetで指定するディレクトリは`*/schema`とするのが良さげ。
パッケージ名変えれたりはしそう。



話を戻して、スキーマの定義をしていきます。この時以下の点を押さえておくとよき。

- IDフィールドは`bigint`で自動的に付与されます
- リレーションに使うforeign key用のフィールドはedgeで定義するので、**スキーマには記述しない**

`User`, `Post`をサンプルとして。

```go
// ent/schema/user.go
import (
	"entgo.io/ent"
	"entgo.io/ent/schema/field"
)

// User holds the schema definition for the User entity.
type User struct {
	ent.Schema
}

// Fields of the User.
func (User) Fields() []ent.Field {
	return []ent.Field{
	// idはbig intで自動生成されるため含めない
		field.String("account_id").Unique(),
		field.Time("created_at"),
		field.Time("updated_at"),
	}
}

```

```go
// ent/schema/post.go
import (
	"entgo.io/ent"
	"entgo.io/ent/schema/field"
)

// Post holds the schema definition for the Post entity.
type Post struct {
	ent.Schema
}

// Fields of the Post.
func (Post) Fields() []ent.Field {
	// 外部キーとなるフィールドは含めない
	return []ent.Field{
		field.String("title"),
		field.String("body"),
		field.Time("created_at"),
		field.Time("updated_at"),
	}
}
```

基本`NOT NULL`制約がつくっぽいです。

次に、実際にスキーマからコードを生成します。

```shell
# go generate <genetate.goがあるディレクトリのpath>
$ go generate ./ent
```

`ent`ディレクトリがエライコッチャになります。

```planetext
ent
├── client.go
├── config.go
├── context.go
├── ent.go
├── enttest
│   └── enttest.go
├── generate.go
├── hook
│   └── hook.go
├── migrate
│   ├── migrate.go
│   └── schema.go
├── mutation.go
├── post
│   ├── post.go
│   └── where.go
├── post.go
├── post_create.go
├── post_delete.go
├── post_query.go
├── post_update.go
├── predicate
│   └── predicate.go
├── profile
│   ├── profile.go
│   └── where.go
├── profile.go
├── profile_create.go
├── profile_delete.go
├── profile_query.go
├── profile_update.go
├── runtime
│   └── runtime.go
├── runtime.go
├── schema
│   ├── post.go
│   ├── profile.go
│   ├── tag.go
│   └── user.go
├── tag
│   ├── tag.go
│   └── where.go
├── tag.go
├── tag_create.go
├── tag_delete.go
├── tag_query.go
├── tag_update.go
├── tx.go
├── user
│   ├── user.go
│   └── where.go
├── user.go
├── user_create.go
├── user_delete.go
├── user_query.go
└── user_update.go
```

ただし、これ以降も編集するのは、`schema`ディレクトリ以下のみとなります。


### migrate

一回DBにスキーマを作ってみます。

```go
// cmd/main.go
package main

import (
	"context"
	"log"
	"net/http"

	"github.com/akubi0w1/ent-sample/ent"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	dbClient, err := ent.Open("mysql", "worker:password@tcp(db:3306)/main")
	if err != nil {
		log.Fatalf("failed to open connetion to mysql: %v", err)
	}
	defer dbClient.Close()

	// ここでmigrationしてくれる
	if err := dbClient.Schema.Create(context.Background()); err != nil {
		log.Fatalf("failed to create schema resources: %v", err)
	}

	// 雑にサーバを動かしているので気にせずに...
	http.ListenAndServe(":8080", nil)
}
```

ちなみに現状で作成されているDBのER図は以下な感じ。孤島。

![](https://64.media.tumblr.com/8e46daf79829f4cc638f4f70b6d7156c/77c86d5077b5095a-b9/s540x810/a1db87529077798db17201d7eb140b7f7de25c2b.png)

データの追加もできます。以下な感じで。

```go
...
now := time.Now()
u, err := dbClient.User.
	Create().
	SetCreatedAt(now).
	SetUpdatedAt(now).
	SetAccountID("accountID").
	Save(context.Background())
if err != nil {
	log.Printf("failed to create user: %v", err)
}
log.Println("user created: ", u)
```

### add edge

次に、edgeを追加していきます。edgeはまあrelationです。
entityをedgeで結んでrelationを作っていきます。
edgeには名前をつける必要がありまして、、、今回作るedgeは以下です。矢印で相互に結んであります。赤字で書いてあるのは多重度です。

![](https://64.media.tumblr.com/ebdb1afbe79e1fd5285ce4a51fe364c5/77c86d5077b5095a-a0/s540x810/1d77fff47b82c37c2fac0b81f27eebb97ae2deb6.png)

`post`と`user`を例に書いていきます。

まず、`user`にてedgeを定義します。

```go
// ent/schema/user.go

// Edges of the User.
func (User) Edges() []ent.Edge {
	return []ent.Edge{
		edge.To("posts", Post.Type),
	}
}
```

ただこれだけだと、`post`側から`user`を参照することができません。edgeは一方向にしか山椒を作らないらしい。

> sqlでいうと、postをselectする時に、inner joinでuserも一緒に持ってくる、みたいなことができない。

そこで、inverse edgeを作成します。これを定義することで、`post`から`user`を参照することができます。

```go
// ent/schema/post.go

// Edges of the Post.
func (Post) Edges() []ent.Edge {
	return []ent.Edge{
		edge.From("author", User.Type).
			Ref("posts"). // Toで指定したedgeName
			Unique(),
	}
}
```

`post`に対して`user`は一意に定まるので、`Unique()`によって1:nなedgeにします。

このようなedgeをrelationの数だけ作成します。作成し終わったら、コードを生成します。

```shell
$ go generate ./ent
```

出来上がったDBのER図はこちら。

![](https://64.media.tumblr.com/b109c4fb7f1764ab4878c1707e0a2618/77c86d5077b5095a-c9/s540x810/9efa1900c47c565f24262231d35f318240ea652e.png)

最初に想定していた形になりましたー。

### 気づいたこととか

#### どちらをTo, Fromにするのか。

n:nのときはどちらでもいい気がしています。それ以外の時は注意が必要。

とりあえず**Fromを指定したスキーマに外部キーがつく**ことを覚えておけば大丈夫。
なので、以下でやるとそれっぽい形になると思います。

- n:n -> どちらでも構わない
- 1:n -> 1=to, n=from
- 1:1 -> 外部キーを持たせたいnodeをfromにする

#### n:nを作った時の複合キー

ちゃんとduplicatieを検知してくれてました。

#### 1スキーマに同名のedgeは作成不可

一つのスキーマに追加するedgeで、同名のものがあるとエラーが出ます。(refで参照できないからかな)


### test

entを使ったコードのテストを書いてみます。ドキュメントは[こちら](https://entgo.io/docs/testing)。

テスト対象のコードは以下。

```go
// mysql/user.go
func Create(cli *ent.Client, ctx context.Context, accountID string, now time.Time) (int, error) {
	u, err := cli.User.Create().
		SetAccountID("accountID").
		SetCreatedAt(now).
		SetUpdatedAt(now).
		Save(ctx)
	if err != nil {
		return 0, err
	}
	return u.ID, nil
}

```

テストコードは以下。

```go
// mysql/user_test.go
import (
	"context"
	"testing"
	"time"

	"github.com/akubi0w1/ent-sample/ent/enttest"
	_ "github.com/mattn/go-sqlite3"
	"github.com/stretchr/testify/assert"
)

func TestUser_Create(t *testing.T) {
	ctx := context.Background()
	t.Run("success", func(t *testing.T) {
		client := enttest.Open(t, "sqlite3", "file:ent?mode=memory&cache=shared&_fk=1")
		defer client.Close()
		
		dummyTime := time.Date(2021, 4, 15, 0, 0, 0, 0, time.UTC)
		expected := 1
		
		out, err := Create(client, ctx, "accID", dummyTime)
		assert.Equal(t, expected, out)
		assert.Nil(t, err)
	})
}
```

生成されたコードの中にclientを作成してくれるコードが混ざっているらしいので、それを使います。

sqliteでやってるけど、mysqlのドライバでなんとかならんかなこれ...

## まとめ

生成されるファイルがめちゃめちゃ多いが、スキーマの管理が割とやりやすいかなーと。常にSQL生書きしていた人間にとっては驚き。

ただこれは慣れかもしれないけど、edgeの設定が慣れない...。どっちをToにするかとか、edgeの名称どうするかなどに頭を悩ませがちだった。

## ref

- [GoのORM「ent」の話 -zenn](https://zenn.dev/masamiki/articles/83a8db3f132fcb1c48f0)
- [ent docs](https://entgo.io/docs/getting-started/)
- [ent -godoc](https://pkg.go.dev/entgo.io/ent)