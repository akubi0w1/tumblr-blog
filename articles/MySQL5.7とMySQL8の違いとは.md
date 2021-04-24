date: 2021/04/13


# MySQL5.7とMySQL8の違いとは

ざっくりまとめ

## 8.0でできるようになったこと

- CTE(Common Table Expressions)
	- With句が使用可能
- Window関数が使用可能
- 権限管理がロール単位で可能
	- ユーザごとの管理より粒度が細かい
- ドキュメントストアが追加された
	- NoSQL DBがが追加されたイメージ
- JSONサポート強化
	- JSON -> RDBテーブルに変換できる関数などの追加
- GIS(Geographic Information System): SRS(Spatial Reference Systems)をサポート
	- 位置管理...？
- デフォルトcharsetがutf8mb4
	- 絵文字とかに対応しているcharsetだね
- 降順インデックスの追加
- インビジブルインデックス(不可視索引)


## 8.0で廃止になったこと

- クエリーキャッシュ
	- SQLのクエリと結果をキャッシュして、同じ文ならキャッシュを使う、っていうのが廃止
- 古いTIME型、TIMESTAMP型の廃止


## ref

- https://qiita.com/kota-miyasaka/items/65a6a1b7bed24771c01d
- https://www.mysql.com/jp/products/enterprise/database/
- http://nippondanji.blogspot.com/2018/05/mysql-80.html
- http://blog.s-style.co.jp/2018/07/2123/


