date: 2021/03/13

# Trello Butlerがめっちゃ便利だったのでログ

タスクをちゃんと整理しようと思ってTrelloのこと調べてたら便利な機能が。。。
ちょっといじくったログ

## Trello Butlerとは

ボード、カードなどの操作を自動化できる便利な機能。

- Rulesでカードの移動などをトリガーとした操作の自動化
- Card Buttonでカードに対する操作を追加
- Board Buttonでボードに対する操作を追加
- Calendarで定期的な操作を追加
- Due Dateで期限をトリガーとした操作を追加

などなど。色々なことが自動化できちゃうぞというもの。
コーディングは一切必要なく、英語が読めればサクサク設定できる。

## 自分が追加したもの

### Rules

#### 1. カードがDoneリストに移動したとき

```
when a card is moved into list "Done" by anyone, check all the items in all the checklists on the card, mark the due date as complete, and set date custom field "DoneAt" to now
```

- カードがDoneリストに移動したとき、
	- カードの、全てのチェックリストの全てのアイテムにチェックする
	- 期限を完了とする
	- カスタムフィールド(power-up)の`DoneAt`を現在時刻に設定する

#### 2. カードがDoneリストから移動されたとき

```
when a card is moved out of list "Done" by anyone, clear custom field "DoneAt"
```

- カードがDoneリストから移動されたとき
	- カスタムフィールドの`DoneAt`を未設定状態に戻す

#### 3. カードがCurrentIntegrationに移動したとき

```
when a card is moved into list "Current Integration" by anyone, set due the next friday
```

- カードがCurrent Integrationリストに移動したとき
	- 期限を次の金曜日にする

### Calendar

#### 1. 毎週土曜9時

```
every saturday at 9:00 am, rename list "Done" to "Done {isodate}", archive list "{newlistname}", and create a new list named "Done" in the bottom position
```

- 毎週土曜日の9時に
	- Doneリストを`Done {現在時刻}`にリネーム
	- リネームしたリストをアーカイブ
	- リストの末尾に新しくDoneリストを作成

## まとめ

ちょっとだけいじくってみたけど、めちゃ便利だなと。ラベルつけたり、カードのアサインとかコメントつけたりもできるらしく、活用していければ...まあ個人規模ではあまり使わなそうだったり。
ただ、リストの名前やフィールド名などをボードで変更した場合、同期を取ってくれるわけではないので、コマンドを変えに行く必要がある。

## ref:

- https://blog.trello.com/ja/butler-power-up-trello-automation
- https://help.trello.com/article/1139-command-libraries
- https://help.trello.com/article/1157-variables