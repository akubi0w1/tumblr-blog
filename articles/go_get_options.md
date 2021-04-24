date: 2020/08/04

## go get options

### go get

開発してるモジュールの依存関係を解決してくれて、必要なモジュールのビルドやインストールをやってくれる。

#### go installとの違いとは...

go install は importで指定されているパッケージのコンパイルとインストールを行ってくれる。

`go get`で`-d`つけたときに必要になる感じかな...

### -t

パッケージのテストに必要なモジュールを考慮してgetを行う。


### -d

パッケージのビルドに必要なコードをダウンロードする。
必要な依存関係を含んでいるが、ビルドとインストールがなされない。


### -u

パッケージの依存関係を提供しているモジュールを更新する。
新しいマイナーバージョンとか、リリースがあった場合、それを使う。


### -v

コンパイル時、パッケージの名前を表示してくれる。
`get`だけでなく、`install`, `list`, `build`,`clean`,`run`,`test`で有効。


## 参考

- [https://golang.org/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them](https://golang.org/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them)
