date: 2021/01/20


# gitのこと

## stash

変更退避

```shell
// 変更退避
$ git stash [save "message"]

// addしていないものだけ退避
$ git statsh -k

// stashの一覧を見る
$ git stash list

// 退避した作業を戻す
$ git stash apply stash@{n}

// 退避した作業を消す
$ git stash drop stash@{n}

// 退避した作業を戻して、stashから消す
$ git stash pop stash@{n}

// 退避した作業の詳細を見る
$ git stash show stash@{n} [-p]
```

## commit

### コミットの取り消し

```shell
$ git reset --hard HEAD^
```

- `--hard`: ワークディレクトリの内容も消しとばす
- `--soft`: ワークディレクトリの内容は残し、コミットだけを消す

- `HEAD^`: 直前のコミットを表す
- `HEAD~{n}`: n個前のコミットを表す

### コミットの打ち消し

作業ツリーの指定のコミット時点の状態まで戻す。
コミットをなかったことにはせず、古いコミットを再コミットする感じ。

```shell
$ git revert {コミットのハッシュ}
```

コミットのハッシュは以下からもってくる

```shell
$ git log
$ git reflog
```

### コミットの上書き

```shell
$ git commit --amend
```

コミットメッセージを変更したい時とか使える。

## 参考
- https://qiita.com/shuntaro_tamura/items/06281261d893acf049ed
- https://qiita.com/chihiro/items/f373873d5c2dfbd03250