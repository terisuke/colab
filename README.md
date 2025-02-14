# colab

### 概要
このシステムは、[ハローワークのHP](https://www.hellowork.mhlw.go.jp/index.html)から[求人情報検索](https://www.hellowork.mhlw.go.jp/kensaku/GECA110010.do?action=initDisp&screenId=GECA110010) に遷移して、自動的に福岡県福岡市博多区の求人情報を検索して会社名、代表者名(あれば)、業種、メールアドレス(あれば）を取得してGoogleスプレッドシートに自動転記するシステムです。

### 修正したいところ
~~現在このシステムは情報は取得できますが、ページ遷移できず50件までしか情報を取得できません。~~
~~もし50件以上の情報を取得したいとしたときに自動的に下のページ遷移を行うことができるようにして下さい。~~

~~![スクリーンショット 2025-02-14 11 33 00](https://github.com/user-attachments/assets/53bdec6b-bcc3-4a9b-82cd-ab9d0547c09c)~~
~~↑ここ~~
【13:20】解決しました。

### 実行の仕方
1.[ここ](https://colab.google/)を開く
2.ログインとかしたらjob_search.ipynbをコピペしてまずはそのまま試してみる(左側の再生ボタンかcmf+enter)
3.スプシとの連携します？って言われるからOKしてログインする
4.できたスプシを[ここ](https://docs.google.com/spreadsheets/u/0/?tgif=d) から確認して中身を見る
5.以下の部分を好きな数字に書き直して実行
```
# 求人情報を取得
all_results = []
max_links = 10 #ここを100に書き直す
first_page = True  # 初回フラグ
```
