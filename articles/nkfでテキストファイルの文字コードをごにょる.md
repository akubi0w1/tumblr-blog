date: 2021/04/18

# nkfでテキストファイルの文字コードをごにょる

## 確認

```shell
$ nkf --guess {file path}
$ nkf -g {file path}
```

## 変更

```shell
$ nkf -w --overwrite {filepath}
```

options

| option | description |
|:--|:--|
| -w | UTF-8で出力 |
| -e | EUC-JPで出力 |
| -s | shift-jisを出力 |