date: 2020/09/10

# terraformについて勉強

## terraform is 何

インフラなどの構成の管理や構築を自動化するツール。

設定ファイルを基にリソースを制御する。設定ファイルの拡張子は`.tf`。

## 設定ファイルについて

### provider

認証情報やリージョンなどの設定を書く。`provider`ブロックで定義。

```
// awsなら
provider "aws" {
	access_key = ""
	secret_key = ""
	region = ""
}
```

### resource

作成するリソースの情報を書く。`resource`ブロックで定義。
リソースの種類はプロバイダごとに決まっている。リソース名は自由に決められる。

```
resource "リソースの種類" "リソース名" {
	設定項目 = "値"
	...
}
```

他のリソースの設定値を参照したい場合は、`${リソースの種類.リソース名.属性名}`で参照できる。

### data

resourceセクションのパラメータにAPIから取得した値を渡すことが可能になる。`data`ブロックで定義？

### variable

`variable`ブロックで変数を定義できる。空ブロックで変数の宣言を行う。デフォルト値を設定する場合は、ブロックに`default`で定義する。
値の参照は、`var.VARIABLE_NAME`で行う。`${}`を使うと文字列に値を埋め込める。

```json
// awsなら
variable "access_key" {}
variable "secret_key" {
	default = "SECRET_KEY"
}

provider "aws" {
	access_key = "${var.access_key}"
	secret_key = "${var.secret_key}"
	region = ""
}
```

#### 値を渡す

##### 1. コマンド

以下な感じで、`terraform`コマンドを打つ時に`-var`で与える。

```shell
$ terraform apply -var 'access_key=ACCESS_KEY'
```

##### 2. 環境変数

`TF_VAR_変数名`って形式の環境変数を与えておくと自動で値を渡してくれる。

```shell
$ export TF_VAR_access_key="ACCESS_KEY"
```

みたいな。

##### 3. ファイル定義

推奨されている方法。

`.tfvars`って拡張子のファイルを作って、そこに変数を定義する方法。
`terraform`コマンドを打つ時に、`-var-file *.tfvars`ってオプションをつければ読んでくれる。
`terraform.tfvars`ってファイル名ならオプションなしでも自動で読んでくれそう。

ファイルはこんな感じで。

```
access_key = "ACCESS_KEY"
```

### output

環境の構築結果をコンソールに出力したい時に使う。

```json
output "属性の説明" {
	value = "${リソースの種類.リソースの名前.属性名}"
}
```

## コマンドについて

### インストール

macならbrewでいける。

```
$ brew install terraform
$ brew install tfenv
```

### tfenv

```shell
// version or latest
$ tfenv install {version}

// 使えるバージョンの確認
$ tfenv list

// 環境の切り替え
$ tfenv use {version}
```


### init

ワークスペースの初期化。

```shell
$ terraform init
```

### plan

```shell
$ terraform plan
```

設定ファイルに誤りがないかや、どんな変更がなされるのかを確認するためのコマンド。
構文のエラーやパラメータの誤りは検知するが、値そのものの正しさは検知しない。

`plan`を実行すると、リソースの状態を表すjsonファイル(`terraform.tfstate`)が作られる。

### apply

```shell
$ terraform apply
```

実際にリソースを作成する。`terraform.tfstate`も更新される。

`show`すると、`tfstate`を見やすい感じに整形してくれる。

```shell
$ terraform show
```

### destroy

```shell
$ terraform destroy

// 差分を見る
$ terrafrom plan -destroy
```

設定ファイルにあるリソースを一式削除する。差分は`plan -destroy`で見る。

## workspace

### workspace is 何

環境を複数用意する場合に、terraform側でstateを分けて管理できる機能。


### コマンド

```shell
// 作成
$ terraform workspace new {workspace_name}

// 切り替え
$ terraform workspace select {workspace_name}

// 現在のworkspaceを確認
$ terraform workspace show

// workspaceの一覧
$ terraform workspace list

// 削除
$ terrafrom delete {workspace_name}
```

workspaceが作成されると、`terraform.tfstate.d`ってディレクトリが作られて、その中に、workspaceの名前がついたディレクトリが作成される。その中に、`terraform.tfstate`が保管されるようになる。

なので、stateを確認したい場合は、`terraform.tfstate.d/{workspace_name}/terraform.tfstate`を見る感じになる。

## 参考
- [1. Terraform について - Enterprise Cloud](https://ecl.ntt.com/documents/tutorials/terraform/rsts/Terraform/about_terraform.html)
- [AWSでTerraformに入門 - Developers.IO](https://dev.classmethod.jp/articles/terraform-getting-started-with-aws)
- [Terraform Workspacesの基礎と使い方について考えてみた！ #AdventCalendar - Developers.IO](https://dev.classmethod.jp/articles/how-to-use-terraform-workspace/)


