date: 2021/03/13

# zsh + preztoでzshをはじめる

`zsh`を使い始める前にterminalを綺麗にしてモチベをあげる。

## 現状確認

```shell
# 現在使っているシェルの確認
$ echo $SHELL

# 利用可能なシェルを確認
$ cat /etc/shells

# zshがなければ以下でインストール
# zshのインストール
$ brew install zsh
```

## zshのpathを通す

```shell
# zshのpathを確認
$ cat /etc/shells
# 確認したzshのpathを設定
$ sudo sh -c "echo '/bin/zsh' >> /etc.shells"
$ chsh -s /bin/zsh
```

## preztoを導入する

REF: [sorin-ionescu/prezto](https://github.com/sorin-ionescu/prezto)

### preztoとは

zshの設定フレームワーク。CLIをエイリアス、関数、テーマとかをよりよくするためのもの。

### install

[GitHubのリポジトリ](https://github.com/sorin-ionescu/prezto)を元にインストールする。

```shell
$ git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"

# 以下の行は4行まとめて実行
$ setopt EXTENDED_GLOB
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
  ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done
```

この時点でホームディレクトリ以下のファイルが作成されている。

- .zlogin
- .zlogout
- .zprezto/
- .zpreztorc
- .zprofile
- .zshenv
- .zshrc

次に、`~/.zshrc`に以下を追記する。

```
autoload -Uz promptinit
promptinit

# 以前の.zshrcに設定があった場合はそれをコピペ
```

設定をロードする。

```shell
$ source ~/.zshrc
```

ここまでやったら、promptがカラフルになる。

### set theme prezto

いい感じのテーマにする。

```shell
# テーマの確認
$ prompt -l

# テーマのプレビュー
$ prompt -p

# テーマの保存
$ prompt -s テーマ
```

powerlineテーマのインストール

```shell
# install yard
$ git clone https://github.com/skwp/dotfiles ~/.yadr
$ cd ~/.yadr && rake install

# create a ~/.secrets file (required by yard)
$ touch ~/.secrets

# install the prompt
$ curl https://raw.github.com/davidjrice/prezto_powerline/master/prompt_powerline_setup > ~/.zsh.prompts/prompt_powerline_setup

# install solarized
$ git clone https://github.com/altercation/solarized
$ cd solarized

# in iTerm2 open preferences 
#   profiles > default > colours > load presets > Solarized Dark
#   profiles > default > terminal > report terminal type > "xterm-256color"

$ echo "prompt powerline" > ~/.zsh.after/prompt.zsh
```

```
# ~/.zpreztorc
zstyle ':prezto:module:prompt' theme 'powerline'
```

### vscodeのterminalを変更

```
// setting.json
{
	"terminal.integrated.shell.osx": "/bin/zsh",
	...
}
```

### 追加でなんか

vim開こうとしたらこんなのが出るようになった
`The legacy SnipMate parser is deprecated. Please see :h SnipMate-deprecate.`

.vimrcを作り直したらなんとかなったw

### zshrc


こんなのが出るようになった。
```
zsh compinit: insecure directories, run compaudit for list.
Ignore insecure directories and continue [y] or abort compinit [n]? y
```

```
$ compaudit
$ chmod 755 対象ファイル
```
で解決できる

## Powerlevel10k

[Powerlevel10k](https://github.com/romkatv/powerlevel10k)を導入する。

### Powerlevel10kとは

zshのテーマ。動作が速く、設定が柔軟にできるテーマ。

### install

`Powerlevel10k`を導入する前に、オススメされているフォント(`Meslo Nerd Font`)をインストールしておく。フォントをインストールすることで、アイコン、セパレータなどの表示が増えるため、プロンプトをよりおしゃんにできる。
フォントをインストールしなくても、`Powerlevel10k`をインストールすることは可能。

```shell
#!/bin/sh
fonts=("Regular" "Bold" "Italic" "Bold Italic")
for ((i = 0; i < ${#fonts[@]}; i++)); do
    if [ ! -e "${HOME}/Library/Fonts/MesloLGS NF ${fonts[$i]}.ttf" ]; then
        _font=`echo ${fonts[$i]} | sed -e 's/ /%20/g'`
        wget -P ${HOME}/Library/Fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20${_font}.ttf
    fi
done
```

`Powerlevel10k`をインストールする。せっかくなのでpreztoを使って導入する。

```shell
# ~/.preztorc
# 以下を追記
zstyle :prezto:module:prompt theme powerlevel10k
```

zshを再起動する。

```shell
$ zsh
```

以下のようなのメッセージがでると、成功している。

```
Powerlevel10k configuration wizard has been aborted. It will run again next time unless
you define at least one Powerlevel10k configuration option. To define an option that
does nothing except for disabling Powerlevel10k configuration wizard, type the following
command:

  echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >>! ~/.zshrc

To run Powerlevel10k configuration wizard right now, type:

  p10k configure
```

`Powerlevel10k`にはセットアップウィザードがあり、これでだいたい設定できる。

```shell
$ p10k configure
```

セットアップが終わるとterminalがいい感じになる。以降のカスタムは、`~/.p10k.zsh`を編集して行う。

## 別で、オススメされているfontのDL

`Meslo Nerd Font`がオススメらしい

## fzfを導入する

ref: [junegunn/fzf](https://github.com/junegunn/fzf)

### fzfとは

コマンドラインでめっちゃ便利な検索ができるようになる。
文字の入力に対してインタラクティブに結果を反映してくれる。コマンドの履歴とか、gitのコミットとかも探せる。

### install

[公式リポジトリ](https://github.com/junegunn/fzf)を参考に行う

```shell
$ git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
$ ~/.fzf/install
```

以降の設定は、`~/.fzf.zsh`で行う。

### ショートカットの変更

検索を行う際のキーバインドを変更する。

```shell
# ~/.fzf.zsh
# ref: https://github.com/junegunn/fzf/wiki/Configuring-fuzzy-completion
# tabキーは通常の挙動、Ctrl + Tでfzfの検索を開ける
export FZF_COMPLETION_TRIGGER=''
bindkey '^T' fzf-completion
bindkey '^I' $fzf_default_completion
```

### プロンプトの色をいじる

プロンプトの色を一部だけちょっといじりたい

ref: https://github.com/romkatv/powerlevel10k/blob/master/README.md#how-do-i-change-prompt-colors

## ref

- https://github.com/sorin-ionescu/prezto
- https://progriro.net/zsh-prezto/
- https://dev.classmethod.jp/articles/zsh-prezto/
- https://github.com/davidjrice/prezto_powerline
- https://dev-yakuza.posstree.com/environment/mac-iterm-zsh/
- [補完について](https://qiita.com/relastle/items/237ca33ac5a605702e25)
