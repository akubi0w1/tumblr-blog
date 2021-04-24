date: 2020/08/04

## mysql DDL で 外部キー設定時のチェックを外す


### やり方

DDLの前に以下を突っ込む。

```sql
SET foreign_key_checks=0;
```


### 解説？
MySQLにて、以下クエリを実行しようとすると、こんな感じのエラーを吐かれます。

```sql
-- DBが作られている状態で、

CREATE TABLE posts (
	id int NOT NULL AUTO_INCREMENT,
	title varchar(100) NOT NULL,
	body text NOT NULL
	user_id int NOT NULL
	PRIMARY KEY (id),
	CONSTRAINT
		FOREIGN KEY (user_id)
		REFERENCES users (id)
		ON UPDATE CASCADE
		ON DELETE CASCADE
);

CREATE TABLE users (
	id int NOT NULL AUTO_INCREMENT,
	name varchar(30) NOT NULL,
	PRIMARY KEY (id)
);
```

```
// エラー
ERROR 1215 (HY000): Cannot add foreign key constraint
```

`posts`テーブルの作成時に、FKの参照先(`users.id`)が存在しないので、設定できないよって言われる。

こういった外部キー周りのチェックを無効にするために、DDLの前に設定を変更しておく。

```sql
SET foreign_key_checks=0;
```

### 参考

- [https://stackoverflow.com/questions/15534977/mysql-cannot-add-foreign-key-constraint](https://stackoverflow.com/questions/15534977/mysql-cannot-add-foreign-key-constraint)
