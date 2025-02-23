# ハローワーク求人情報スクレイピング＆役員情報取得ツール

本レポジトリでは、ハローワークの求人情報をスクレイピングし、その取得結果をGoogleスプレッドシートに保存する Python スクリプトを提供しています。また、スクレイピングした企業名を元に Perplexity API と Gemini API から役員情報(役職・名前)を取得し、Hunter.io API で推定メールアドレスを得て同じくスプレッドシートに書き込みます。

## 概要

1.  **ハローワーク求人サイトのスクレイピング**
    *   福岡県の博多区を対象とし、企業名・代表者名・産業分類・メールアドレス・求人詳細URL などを収集します。
    *   収集した求人情報は Google スプレッドシートに保存します。
2.  **役員情報の取得 (Perplexity + Gemini)**
    *   企業名を用いて Perplexity API で情報を検索し、その応答から役員名・役職を Gemini API で抽出します。
    *   役員名の日本語表記とローマ字を取得し、候補が見つからない場合はスクリプト内で用意している簡易パーサーで取得します。
3.  **Hunter.io API を使ったメールアドレス推定**
    *   役員の「ローマ字の名・ローマ字の姓・ドメイン」をキーに、Hunter.io API の Email Finder を呼び出して最も可能性の高いメールアドレスを推定します。
    *   推定メールアドレスは、役員情報シートに「Hunter推定メールアドレス」として記録します。

## 前提条件

本スクリプトは以下のサービスやモジュールに依存しています。事前にアカウントや環境を用意してください。

1.  **Google Colab またはローカルでの実行**
    *   Colab やローカル環境のいずれでも動作しますが、Google スプレッドシートを操作するため、Google アカウントの認証が必要です。
2.  **Perplexity API キー**
    *   Perplexity.ai の API を利用するための認証キーが必要です。
    *   スクリプトの冒頭にある `PERPLEXITY_API_KEY` に設定してください。
3.  **Gemini (Generative Language) API キー**
    *   Google Generative Language API (Gemini) のキーが必要です。
    *   スクリプトの冒頭にある `GEMINI_API_KEY` に設定してください。
4.  **Hunter.io API キー**
    *   役員メールアドレスを推定するために必須です。
    *   スクリプト冒頭の `os.environ["HUNTER_API_KEY"] = "xxxxxxxxxxxxxxxxxxxxx"` の部分で設定します。
    *   または `%env HUNTER_API_KEY=xxxxxx` のようにノートブック上やローカル環境で事前に設定しておき、`getenv("HUNTER_API_KEY")` から参照してください。
5.  **必要な Python ライブラリ**
    *   `selenium`, `gspread`, `google-auth`, `beautifulsoup4`, `lxml`, `spacy`, `requests`, `pykakasi`
    *   コードの先頭に `!pip install ...` があるため、Colab で実行する場合は自動インストールされます。
6.  **Google スプレッドシートへの認証**
    *   `google.colab.auth` および `google.auth.default` を使い、Colab 上で「Googleアカウントへの認証」を行います。
    *   ローカル環境の場合はサービスアカウントキーなど別の方法が必要になります。

## 使い方

以下の手順でスクリプトを実行し、成果物を得ることができます。

1.  **Colab もしくはローカルにスクリプトをコピー**
    *   ノートブック ( `.ipynb` ) の形であれば Colab にアップロード、もしくは同じ環境にファイルを配置します。
2.  **環境変数の設定 (Hunter.io APIキーなど)**
    *   ノートブックや実行環境にて Hunter.io のキーをセットしてください。

    ```bash
    %env HUNTER_API_KEY=your_hunter_api_key
    ```

    *   もしくはスクリプト冒頭の `os.environ["HUNTER_API_KEY"] = "xxxxxxxxxxx"` の部分を直接編集して実際のキーを記入してください。

3.  **Perplexity/Gemini API キーの設定**
    *   スクリプト上部の `PERPLEXITY_API_KEY`, `GEMINI_API_KEY` 変数に実際のキーを記載してください。
4.  **Google 認証 (Colab利用時)**
    *   実行すると最初に `auth.authenticate_user()` が呼ばれ、Google アカウントへの認証を求められます。
    *   ブラウザ経由で認証し、トークンを取得してください。
5.  **実行**
    *   Colab であれば「セルをすべて実行」ボタンを押すか、上から順番にセルを実行します。
    *   ローカルの場合は `python script_name.py` で実行できます (ChromeDriver など事前準備が必要)。
6.  **出力確認**
    *   実行が完了すると、指定した Google スプレッドシート (`SPREADSHEET_NAME`) に新規シート「求人情報」と「役員情報」が作成され、スクレイピング結果・役員情報・Hunter.ioで推定されたメールアドレスが記入されます。
    *   役員情報の「Hunter推定メールアドレス」列にメールアドレス(または N/A) が書き込まれます。

## コードの流れ

1.  **ライブラリのインストール＆インポート**
    *   `!apt-get update -qq` 等で ChromeDriver をインストールし、Python の依存ライブラリを入れます。
    *   `import` 文で必要なモジュールを読み込みます。
2.  **設定**
    *   `PERPLEXITY_API_KEY`, `GEMINI_API_KEY` などの変数を設定します。
    *   `SPREADSHEET_NAME`, `JOB_SHEET_NAME`, `OFFICER_SHEET_NAME` の各名前なども自由に変更可能です。
3.  **Google 認証 & スプレッドシート準備**
    *   `auth.authenticate_user()`, `default()` により Google アカウントの認証を行います。
    *   `get_or_create_sheet()` でスプレッドシートとシートを取得（なければ新規作成）します。
4.  **Chrome (headless) の起動**
    *   `webdriver.Chrome(options=...)` でヘッドレスブラウザを起動します。
5.  **ハローワークからの求人情報スクレイピング**
    *   `scrape_job_data()` 関数で、福岡県博多区の企業情報を取得し、最大 `MAX_LINKS` 件まで抽出します。
    *   取得したデータは「会社名」「代表者名」「メールアドレス」「産業分類」などをまとめて返します。
6.  **Perplexity からの役員情報取得**
    *   各企業ごとに `get_officers_from_perplexity()` を呼び出し、APIから得られたテキストを受け取ります。
    *   テキストからGemini APIまたはフォールバックパーサーで役員情報（役職・名前）を抽出します。
7.  **Hunter.io で役員メールアドレス推定**
    *   役員名のローマ字 (名・姓) と会社ドメインをキーに `get_hunter_email()` を呼び出します。
    *   正常に取得できれば、そのアドレスを「Hunter推定メールアドレス」列に格納します。
8.  **スプレッドシート書き込み**
    *   求人情報は `JOB_SHEET_NAME` シートに、役員情報は `OFFICER_SHEET_NAME` シートに出力します。

## よくある質問 (FAQ)

*   **Q: レートリミットに引っかかったらどうなりますか？**
    Hunter.io は 1秒間15リクエスト、1分間500リクエストの制限があり、APIが 429 を返す場合があります。現在の実装では自動リトライせず、見つからなかった扱い (N/A) にしています。大量リクエストを送る際は `time.sleep()` を増やすなどの対策を行ってください。

*   **Q: 役員のローマ字がうまく変換されません**
    `pykakasi` の変換が想定外の読みを返す場合があります。正しいローマ字を手動修正したり、より高度な形態素解析を導入するなどカスタマイズが必要です。

*   **Q: 企業によっては複数ドメインを持っていますが、どのドメインが使われますか？**
    スクリプト上では最初の1つのドメインを使い、Hunter.io でメールを検索しています。複数ドメインをすべて試したい場合は、該当箇所を修正してループ処理を追加してください。

*   **Q: Google Colab 以外で実行したい**
    ChromeDriver や Selenium のセットアップを個別に行う必要があります。Colab ではなくローカル環境で行う場合、`pip install selenium gspread ...` を実行し、さらにChromeDriverをシステムにインストールしてください。また、Google スプレッドシート操作にはサービスアカウントキー等を使った認証が必要です。

## ライセンス

本スクリプト自体は特にライセンスを定めていません。利用する外部ライブラリや API はそれぞれのライセンスや利用規約に従ってください。特に Hunter.io, Perplexity.ai, Google Generative Language (Gemini) などの商用サービスは独自の規約がありますので必ずご確認をお願いいたします。
