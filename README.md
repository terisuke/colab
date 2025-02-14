# colab

### 概要
このシステムは、[ハローワークのHP](https://www.hellowork.mhlw.go.jp/index.html)から[求人情報検索](https://www.hellowork.mhlw.go.jp/kensaku/GECA110010.do?action=initDisp&screenId=GECA110010) に遷移して、自動的に福岡県福岡市博多区の求人情報を検索して会社名、代表者名(あれば)、業種、メールアドレス(あれば）を取得してGoogleスプレッドシートに自動転記するシステムです。

### 修正したいところ
現在このシステムは情報は取得できますが、ページ遷移できず50件までしか情報を取得できません。
もし50件以上の情報を取得したいとしたときに自動的に下のページ遷移を行うことができるようにして下さい。

![スクリーンショット 2025-02-14 11 33 00](https://github.com/user-attachments/assets/53bdec6b-bcc3-4a9b-82cd-ab9d0547c09c)
↑ここ

### 実行の仕方
1.[ここ](https://colab.google/)を開く
2.ログインとかしたらjob_search.ipynbをコピペしてまずはそのまま試してみる(左側の再生ボタンかcmf+enter)
3.スプシとの連携します？って言われるからOKしてログインする
4.できたスプシを[ここ](https://docs.google.com/spreadsheets/u/0/?tgif=d) から確認して中身を見る
5.以下の部分を10から100に書き直して試してみる
```
# 求人情報を取得
all_results = []
max_links = 10 #ここを100に書き直す
first_page = True  # 初回フラグ
```
6.多分こんなエラーが出る
```
次のページへの遷移中にエラーが発生しました: Message: no such element: Unable to locate element: {"method":"css selector","selector":"[name="fwListNaviBtnNext"]"}
(Session info: chrome=133.0.6943.98); For documentation on this error, please visit: https://www.selenium.dev/documentation/webdriver/troubleshooting/errors#no-such-element-exception
Stacktrace:
#0 0x56428e900bba <unknown>
#1 0x56428e39e790 <unknown>
#2 0x56428e3efc80 <unknown>
#3 0x56428e3efe01 <unknown>
#4 0x56428e43e944 <unknown>
#5 0x56428e415a7d <unknown>
#6 0x56428e43bccc <unknown>
#7 0x56428e415823 <unknown>
#8 0x56428e3e1a88 <unknown>
#9 0x56428e3e2bf1 <unknown>
#10 0x56428e8ca15b <unknown>
#11 0x56428e8ce0e2 <unknown>
#12 0x56428e8b701c <unknown>
#13 0x56428e8cecd4 <unknown>
#14 0x56428e89b48f <unknown>
#15 0x56428e8ef4f8 <unknown>
#16 0x56428e8ef6c9 <unknown>
#17 0x56428e8ffa36 <unknown>
#18 0x795f20508ac3 <unknown>
```
ここを解決したい

### 今やっていること
1.jon_search.ipynbと求人検索一覧のhtml要素とエラー結果を比較して要素のロジックを確認してみる
![スクリーンショット 2025-02-14 11 43 46](https://github.com/user-attachments/assets/48fcb6c7-1ca9-4c2f-b708-9cb0b4b1f347)
↑二本指でクリックすると「ページのソースを表示」が出てくるので、ここからhtml要素を見ることができる

2.検索結果一覧ページ(例:https://www.hellowork.mhlw.go.jp/kensaku/GECA110010.do など)は一定時間経つor別タブで見ようとすると
```
システムの混雑、または続行不可能なエラーが発生しました。
```
ってエラーになって見れない。
これがページ遷移できない理由の一つかも。
