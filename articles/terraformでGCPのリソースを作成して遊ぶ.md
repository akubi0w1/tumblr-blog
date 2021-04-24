date: 2021/04/23

# terraformでGCPのリソースを作成して遊ぶ

[TOC]

terraformを触ったことがなかったので触って遊んでみる。

## 作ってみるリソースたち

- Cloud Run
	- サーバレスなマネージドコンピューティングプラットフォーム
	- コンテナイメージを元にアプリを動かせる
- Cloud SQL
	- リレーショナルデータベースを扱えるフルマネージドなサービス
	- MySQL, PostgreSQL, SQL Serverを利用できる
- Cloud Storage
	- コンテンツのストレージ
- Container Registry
	- Dokcerイメージのストレージ
- Secret Manager
	- 機密データを保存するDBのようなもの

## GCPコンソールでぽちぽちやること

多分`gcloud`コマンドでもできるけど、とりあえずコンソールでぽちぽちやります。

1. サービスアカウントの作成
	1. role付与
	2. キーの作成
3. APIの有効化

### サービスアカウントの作成

まずはサービスアカウントの作成を行います。サクッとコンソールをつかってやっていく。

1. コンソールの `IAMと管理 > サービスアカウント`より、`サービスアカウントの作成`をクリック
	1. 適当に、`tf-access-account`とかで、作成する。
	2. roleは別につけなくていいかな。後でつけられるし。

次に、作成したサービスアカウントに対してキーを設定します。

1. サービスアカウントの管理画面の`キー`より、`鍵を追加`をクリック
2. JSONを選択して鍵を作成し、キーファイルをダウンロード
3. これを後々使うので保管する

次にroleの設定を行います。

1. `サービスアカウントの管理画面 > 権限 > アクセスを許可`より、`Project > 編集者`で付与。(たぶんもっとミニマムにできる)
2. `IAMと管理 > IAM`にて`Secret Manager 管理者`を追加する。

### APIの有効化

APIを有効化します。[API Library](https://console.cloud.google.com/apis/library)から検索できます。

1. Cloud Run APIの有効化
	1. https://console.cloud.google.com/apis/library/run.googleapis.com
2. Cloud SQL Admin APIの有効化
	1. https://console.cloud.google.com/apis/library/sqladmin.googleapis.com
3. Secret Manager APIの有効化
	1. https://console.cloud.google.com/apis/library/secretmanager.googleapis.com

以上で大体の準備は完了。terraformを書いていくぞー。

## terraformとは

軽く過去に知識整理をしているので、そちらを参考にー。

https://akubi0w1.tumblr.com/search/terraform

## terraformを書くぞ

ファイルの構成は以下な感じにします。

```
terraform-sample/
├── README.md
├── main.tf // リソースの構成をガーッと書いているファイル
├── terraform.tfvars // 変数に入れる値の管理
├── secrets.tfvars // 変数に入れる値の管理(非公開にしたいもの)
├── <SERIVCE_ACCOUNT_KEY>.json // GCPサービスユーザのアクセスキーのファイル
└── variables.tf // 変数の管理
```

コードを載せると長いので、[GitHub](https://github.com/akubi0w1/terraform-sample)を参考にしてください。
ただ、`secrets.tfvars`だけはGitHubに公開してない(非公開の値を入れているため)ので、こちらで補足。

```terraform
# secrets.tfvars
# PROJECT_IDはコンソールのダッシュボードなどから調べることができます
project = "<PROJECT_ID>"

# 先ほどサービスアカウントのキーファイルをDLしたと思うので、そのファイルを指定します
credentials_file = "<SERIVCE_ACCOUNT_KEY>.json"

database_main = {
    instance_name = "sample-cloudsql-instance"
    version = "MYSQL_5_7"
    region = "us-east1"
    db = "main"
    user = "worker"
    password = "Passowrd"
    host = "%"
}
```

## 適用してみるぞ

まずは初期化。必要なプラグインを落っことす。

```shell
$ terraform init
```

terraformコマンドを実行する時に、変数の定義として`valiables.tf`、値の定義として`terraform.tfvars`は自動的に読み込まれますが、そのほかの変数や定義、ファイルなどはオプションで指定する必要があります。なので。

```shell
$ terraform plan -var-file=secrets.tfvars

# 適用する時は
$ terraform apply -var-file=secrets.tfvars

# targetを指定する
# resource=<リソースの種類>.<リソース名> 
$ terraform [plan | apply] -target=resource

# クリーンアップ
$ terraform destroy
```

## ref

- [HashiCorp公式チュートリアル](https://learn.hashicorp.com/terraform?track=gcp#gcp)
- [Google cloud platform provider -hashicorp doc](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [gcp provider releases](https://github.com/hashicorp/terraform-provider-google-beta/blob/master/CHANGELOG.md)
- GCP
	- [Cloud Run](https://cloud.google.com/run/docs?hl=ja)
	- [Cloud SQL](https://cloud.google.com/sql/docs?hl=ja)
	- [Cloud Storage](https://cloud.google.com/storage/docs?hl=ja)
	- [Container Registry](https://cloud.google.com/container-registry/docs)
	- [Secret Manager](https://cloud.google.com/secret-manager/docs)
- Terraform
	- [cloud run service](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/cloud_run_service)
	- [cloud sql database instance](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/sql_database_instance)
	- [cloud sql database](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/sql_database)
	- [cloud sql user](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/sql_user)
	- [cloud storage bucket](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket)
	- [container registry](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_registry)
	- [container registry image](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/container_registry_image)
	- [secret manager secret](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/secret_manager_secret)
	- [secret manager secret version](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/secret_manager_secret_version)