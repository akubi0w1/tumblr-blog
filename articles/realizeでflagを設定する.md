date: 2021/01/20

# realizeでflagを設定する

golangでrealizeを使用して、ホットリロードを有効にした開発をするときのメモ。realizeの設定で、flagを設定してみる。

`schema`部分に`args`を設定する。`--{flagName}={flagValue}`のフォーマットで書けば適用できるっぽい。

```yaml
settings:
  ...
schema:
- name: api
  path: .
  args:
    - --mode=develop
  commands:
    ...
```

ちなみに、値の取り出し方サンプル

```go
modeFlag := flag.String("mode", "production", "run mode. value=[develop, production]")
flag.Parse()
```

## ref
- [Package flag -doc](https://golang.org/pkg/flag/)
- [oxequa/realize -github](https://github.com/oxequa/realize)