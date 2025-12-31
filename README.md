# lxml によるWebスクレイピング

[![Bright Data Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

このガイドでは、Python の `lxml` パッケージを使用して静的および動的コンテンツを解析し、一般的な課題を克服し、データ抽出プロセスを効率化する方法を説明します。

- [Pythonでlxmlを使ったWebスクレイピング](#using-lxml-for-web-scraping-in-python)
- [前提条件](#prerequisites)
- [静的HTMLコンテンツの解析](#parsing-static-html-content)
- [動的HTMLコンテンツの解析](#parsing-dynamic-html-content)
- [Bright Data Proxyとlxmlの併用](#using-lxml-with-bright-data-proxy)

## Pythonでlxmlを使ったWebスクレイピング

Web上では、構造化された階層データは HTML と XML の2つの形式で表現できます。

- **XML** は、あらかじめ用意されたタグやスタイルを持たない基本的な構造です。コーダーが独自のタグを定義して構造を作成します。タグの主な目的は、異なるシステム間で理解可能な標準データ構造を作ることです。
- **HTML** は、事前定義されたタグを持つWebマークアップ言語です。これらのタグには、`<h1>` タグの `font-size` や `<img />` タグの `display` のようなスタイルプロパティが含まれます。HTML の主な機能は、Webページを効果的に構造化することです。

lxml は HTML と XML の両方のドキュメントで動作します。

### Prerequisites

lxml でWebスクレイピングを開始する前に、マシンにいくつかのライブラリをインストールする必要があります。

```sh
pip install lxml requests cssselect
```

このコマンドで以下がインストールされます。

- XML と HTML を解析するための `lxml`
- Webページを取得するための `requests`
- CSS セレクタを使用して HTML 要素を抽出する `cssselect`

### Parsing Static HTML Content

スクレイピングできるWebコンテンツには、主に静的と動的の2種類があります。静的コンテンツはWebページの初回読み込み時にHTMLドキュメントへ埋め込まれているため、スクレイピングが容易です。一方、動的コンテンツは初回読み込み後に継続的に読み込まれたり、JavaScript によってトリガーされたりします。

まず、ブラウザの **Dev Tools** を使って対象となる HTML 要素を特定します。Webページを右クリックして **Inspect** を選択するか、Chrome で **F12** を押すと **Dev Tools** を開けます。

![DevTools in Chrome](https://github.com/bright-jp/web-scraping-with-lxml/blob/main/images/DevTools-in-Chrome-1024x576.png)

画面右側に、ページのレンダリングを担当するコードが表示されます。各書籍データを扱う特定の HTML 要素を見つけるには、ホバーして選択するオプション（画面左上の矢印）を使ってコードを辿ります。

![Hover-to-select option in Dev Tools](https://github.com/bright-jp/web-scraping-with-lxml/blob/main/images/Hover-to-select-option-in-Dev-Tools-1024x587.png)

**Dev Tools** では、次のコードスニペットが表示されるはずです。

```html
<article class="product_pod">
<!-- code omitted -->
<h3><a href="catalogue/a-light-in-the-attic_1000/index.html" title="A Light in the Attic">A Light in the ...</a></h3>
            <div class="product_price">
        <p class="price_color">£51.77</p>
<!-- code omitted -->
            </div>
    </article>
```

`static_scrape.py` という新しいファイルを作成し、次のコードを入力します。

```python
import requests
from lxml import html
import json

URL = "https://books.toscrape.com/"

content = requests.get(URL).text
```

次に、HTML を解析してデータを抽出します。

```python
parsed = html.fromstring(content)
all_books = parsed.xpath('//article[@class="product_pod"]')
books = []
```

このコードは、`html.fromstring(content)` を使って `parsed` 変数を初期化し、HTML コンテンツを階層的なツリー構造へ解析します。`all_books` 変数では XPath セレクタを使用し、Webページから class が `product_pod` のすべての `<article>` タグを取得します。この構文は XPath 式として有効です。

次に、書籍を反復処理してタイトルと価格を抽出します。

```python
for book in all_books:
    book_title = book.xpath('.//h3/a/@title')
    price = book.cssselect("p.price_color")[0].text_content()
    books.append({"title": book_title, "price": price})

```

`book_title` 変数は、`<h3>` タグ内の `<a>` タグから `title` 属性を取得する XPath セレクタで定義されています。XPath 式の先頭にあるドット（`.`）は、デフォルトの開始点ではなく `<article>` タグから検索を開始することを指定します。

次の行では、`cssselect` メソッドを使って class が `price_color` の `<p>` タグから価格を抽出します。`cssselect` はリストを返すため、インデックス（`[0]`）で先頭要素にアクセスし、`text_content()` で要素内のテキストを取得します。

抽出されたタイトルと価格のペアは辞書として `books` リストに追加され、JSON ファイルに容易に保存できます。

次に、抽出データを JSON ファイルとして保存します。

```python
with open("books.json", "w", encoding="utf-8") as file:
    json.dump(books, file)
```

スクリプトを実行します。

```sh
python static_scrape.py
```

このコマンドにより、ディレクトリ内に次の出力を持つ新しいファイルが生成されます。

![static_scrape.py JSON output](https://github.com/bright-jp/web-scraping-with-lxml/blob/main/images/static_scrape.py-JSON-output-1024x695.png)

このスクリプトの全コードは [GitHub](https://gist.github.com/vivekthedev/c1c5f0fb0e23cabfa3fa5c364b939f7c) で参照できます。

### Parsing Dynamic HTML Content

動的コンテンツをスクレイピングするには、[Selenium](https://www.selenium.dev/) をインストールします。

```sh
pip install selenium
```

YouTube は JavaScript によってレンダリングされるコンテンツの代表例です。キーボード操作をエミュレートしてページをスクロールし、[freeCodeCamp.org YouTube channel](https://www.youtube.com/c/Freecodecamp) の上位100本の動画データをスクレイピングしてみましょう。

まず、**Dev Tools** でWebページの HTML コードを確認します。

![FreeCodeCamp page on YouTube](https://github.com/bright-jp/web-scraping-with-lxml/blob/main/images/FreeCodeCamp-page-on-YouTube-1024x576.png)

次のコードは、動画タイトルとリンクを表示する要素を特定します。

```html
<a id="video-title-link" class="yt-simple-endpoint focus-on-expand style-scope ytd-rich-grid-media" href="/watch?v=i740xlsqxEM">
<yt-formatted-string id="video-title" class="style-scope ytd-rich-grid-media">GitHub Advanced Security Certification – Pass the Exam!
</yt-formatted-string></a>
```

動画タイトルは ID が `video-title` の `yt-formatted-string` タグ内にあり、動画リンクは ID が `video-title-link` の `a` タグの `href` 属性にあります。

`dynamic_scrape.py` を作成し、必要なモジュールをインポートします。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys

from lxml import html

from time import sleep
import json

```

ブラウザドライバを定義します。

```python
URL = "https://www.youtube.com/@freecodecamp/videos"
videos = []
driver = webdriver.Chrome()

driver.get(URL)
sleep(3)
```

前のスクリプトと同様に、スクレイピングしたいWeb URL を含む `URL` 変数と、すべてのデータをリストとして格納する `videos` 変数を宣言します。

次に、ブラウザと対話するために `driver` 変数（つまり `Chrome` インスタンス）を宣言します。`get()` 関数はブラウザインスタンスを開き、指定した `URL` へリクエストを送信します。

その後、Webページ上の要素にアクセスする前に `sleep` 関数を呼び出して3秒待機し、ブラウザ内にすべての HTML コードが読み込まれるようにします。

次に、下方向へのスクロールをエミュレートして追加の動画を読み込みます。

```python
parent = driver.find_element(By.TAG_NAME, 'html')
for i in range(4):
    parent.send_keys(Keys.END)
    sleep(3)
```

`send_keys` メソッドは `END` キーの押下をシミュレートしてページ最下部へスクロールし、さらに動画が読み込まれるようにトリガーします。この操作は `for` ループ内で4回繰り返され、十分な数の動画が読み込まれるようにします。各スクロール後に `sleep` 関数が3秒停止し、次のスクロール前に動画が読み込まれる時間を確保します。

次に、動画タイトルとリンクを抽出します。

```python
html_data = html.fromstring(driver.page_source)

videos_html = html_data.cssselect("a#video-title-link")
for video in videos_html:
    title = video.text_content()
    link = "https://www.youtube.com" + video.get("href")

    videos.append( {"title": title, "link": link} )
```

このコードでは、driver の `page_source` 属性から取得した HTML コンテンツを `fromstring` メソッドへ渡し、HTML の階層ツリーを構築します。

その後、CSS セレクタを使って ID が `video-title-link` のすべての `<a>` タグを選択します。ここで `#` 記号は、タグの ID を使って選択することを示します。この選択は、指定条件を満たす要素のリストを返します。

続いて、各要素を反復処理してタイトルとリンクを抽出します。`text_content` メソッドは内部テキスト（動画タイトル）を取得し、`get` メソッドは `href` 属性値（動画リンク）を取得します。

最後に、データは `videos` というリストに保存されます。

次に、データを JSON ファイルとして保存し、driver を閉じます。

```python
with open('videos.json', 'w') as file:
    json.dump(videos, file)
driver.close()
```

スクリプトを実行します。

```sh
python dynamic_scrape.py
```

スクリプト実行後、`videos.json` という新しいファイルがディレクトリに作成されます。

![dynamic_scrape.py JSON output](https://github.com/bright-jp/web-scraping-with-lxml/blob/main/images/dynamic_scrape.py-JSON-output-1024x495.png)

このスクリプトの全コードも [GitHub](https://gist.github.com/vivekthedev/36489fbaf896eb7c06ebb9350dec298a) で参照できます。

### Using lxml with Bright Data Proxy

Webスクレイピングでは、アンチスクレイピングツールやレート制限といった課題に直面することがあります。プロキシサーバーはユーザーの IPアドレス をマスキングすることで役立ちます。Bright Data は信頼性の高いプロキシサービスを提供しています。

まず、無料トライアルに登録して Bright Data からプロキシを取得します。Bright Data アカウントを作成すると、次のダッシュボードが表示されます。

![Bright Data Dashboard](https://github.com/bright-jp/web-scraping-with-lxml/blob/main/images/Bright-Data-Dashboard-1024x461.png)

**My Zones** オプションに移動し、新しい [residential proxy](https://brightdata.jp/proxy-types/residential-proxies) を作成します。これにより、次のステップで必要となるプロキシの username、password、host が表示されます。

次に、`static_scrape.py` を修正し、URL 変数の下に次のコードを追加します。

```python
URL = "https://books.toscrape.com/"

# new
username = ""
password = ""
hostname = ""

proxies = {
    "http": f"https://{username}:{password}@{hostname}",
    "https": f"https://{username}:{password}@{hostname}",
}

content = requests.get(URL, proxies=proxies).text
```

プレースホルダーを Bright Data の認証情報に置き換え、スクリプトを実行します。

```sh
python static_scrape.py
```

このスクリプトを実行すると、前の例で得られたものと同様の出力が表示されます。

このスクリプト全体は [GitHub](https://gist.github.com/vivekthedev/201f994bc14e4dbc7263b03983f917b3) で参照できます。

## Conclusion

Python で lxml を使用すると効率的なWebスクレイピングが可能になりますが、時間がかかる場合があります。Bright Data は、すぐに使える [datasets](https://brightdata.jp/products/datasets) と [Web Scraper API](https://brightdata.jp/products/web-scraper) により、効率的な代替手段を提供しています。

Bright Data を無料でお試しください！