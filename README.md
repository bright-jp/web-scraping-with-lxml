# Web Scraping with lxml

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to use the `lxml` package in Python to parse static and dynamic content, overcome common challenges, and streamline your data extraction process.

- [Using lxml for Web Scraping in Python](#using-lxml-for-web-scraping-in-python)
- [Prerequisites](#prerequisites)
- [Parsing Static HTML Content](#parsing-static-html-content)
- [Parsing Dynamic HTML Content](#parsing-dynamic-html-content)
- [Using lxml with Bright Data Proxy](#using-lxml-with-bright-data-proxy)

## Using lxml for Web Scraping in Python

On the web, structured and hierarchical data can be represented in two formats—HTML and XML:

- **XML** is a basic structure that does not come with prebuilt tags and styles. The coder creates the structure by defining its own tags. The tag’s main purpose is to create a standard data structure that can be understood between different systems.
- **HTML** is a web markup language with predefined tags. These tags come with some styling properties, such as `font-size` in `<h1>` tags or `display` for `<img />` tags. HTML’s primary function is to structure web pages effectively.

lxml works with both HTML and XML documents.

### Prerequisites

Before you can start web scraping with lxml, you need to install a few libraries on your machine:

```sh
pip install lxml requests cssselect
```

This command installs the following:

- `lxml` to parse XML and HTML
- `requests` for fetching web pages
- `cssselect`, which uses CSS selectors to extract HTML elements

### Parsing Static HTML Content

Two main types of web content can be scraped: static and dynamic. Static content is embedded in the HTML document when the web page initially loads, making it easy to scrape. In contrast, dynamic content is loaded continuously or triggered by JavaScript after the initial page load.

To start, use your browser’s **Dev Tools** to identify the relevant HTML elements. Open **Dev Tools** by right-clicking the web page and selecting the **Inspect** option or pressing **F12** in Chrome.

![DevTools in Chrome](https://brightdata.com/wp-content/uploads/2024/08/DevTools-in-Chrome-1024x576.png)

The right side of the screen displays the code responsible for rendering the page. To locate the specific HTML element that handles each book’s data, search through the code using the hover-to-select option (the arrow in the top-left corner of the screen):

![Hover-to-select option in Dev Tools](https://brightdata.com/wp-content/uploads/2024/08/Hover-to-select-option-in-Dev-Tools-1024x587.png)

In **Dev Tools**, you should see the following code snippet:

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

Create a new file named `static_scrape.py` and input the following code:

```python
import requests
from lxml import html
import json

URL = "https://books.toscrape.com/"

content = requests.get(URL).text
```

Next, parse the HTML and extract data:

```python
parsed = html.fromstring(content)
all_books = parsed.xpath('//article[@class="product_pod"]')
books = []
```

This code initializes the `parsed` variable using `html.fromstring(content)`, which parses the HTML content into a hierarchical tree structure. The `all_books` variable uses an XPath selector to retrieve all `<article>` tags with the class `product_pod` from the web page. This syntax is specifically valid for XPath expressions.

Next, iterate through the books and extract titles and prices:

```python
for book in all_books:
    book_title = book.xpath('.//h3/a/@title')
    price = book.cssselect("p.price_color")[0].text_content()
    books.append({"title": book_title, "price": price})

```

The `book_title` variable is defined using an XPath selector that retrieves the `title` attribute from an `<a>` tag within an `<h3>` tag. The dot (`.`) at the beginning of the XPath expression specifies that the search should start from the `<article>` tag rather than the default starting point. 

The next line uses the `cssselect` method to extract the price from a `<p>` tag with the class `price_color`. Since `cssselect` returns a list, indexing (`[0]`) accesses the first element, and `text_content()` retrieves the text inside the element. 

Each extracted title and price pair is then appended to the `books` list as a dictionary, which can be easily stored in a JSON file.

Now, save the extracted data as a JSON file:

```python
with open("books.json", "w", encoding="utf-8") as file:
    json.dump(books, file)
```

Run the script:

```sh
python static_scrape.py
```

This command generates a new file in your directory with the following output:

![static_scrape.py JSON output](https://brightdata.com/wp-content/uploads/2024/08/static_scrape.py-JSON-output-1024x695.png)

All the code for this script is available on [GitHub](https://gist.github.com/vivekthedev/c1c5f0fb0e23cabfa3fa5c364b939f7c).

### Parsing Dynamic HTML Content

To scrape dynamic content, install [Selenium](https://www.selenium.dev/):

```sh
pip install selenium
```

YouTube is a great example of content rendered using JavaScript. Let's scrape data for the top hundred videos from the [freeCodeCamp.org YouTube channel](https://www.youtube.com/c/Freecodecamp) by emulating keyboard presses to scroll the page.

To begin, inspect the HTML code of the web page with **Dev Tools**:

![FreeCodeCamp page on YouTube](https://brightdata.com/wp-content/uploads/2024/08/FreeCodeCamp-page-on-YouTube-1024x576.png)

The following code identifies the elements responsible for displaying the video title and link:

```html
<a id="video-title-link" class="yt-simple-endpoint focus-on-expand style-scope ytd-rich-grid-media" href="/watch?v=i740xlsqxEM">
<yt-formatted-string id="video-title" class="style-scope ytd-rich-grid-media">GitHub Advanced Security Certification – Pass the Exam!
</yt-formatted-string></a>
```

The video title is within the `yt-formatted-string` tag with the ID `video-title`, and the video link is located in the `href` attribute of the `a` tag with the ID `video-title-link`.

Create `dynamic_scrape.py` and import required modules:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys

from lxml import html

from time import sleep
import json

```

Define the browser driver:

```python
URL = "https://www.youtube.com/@freecodecamp/videos"
videos = []
driver = webdriver.Chrome()

driver.get(URL)
sleep(3)
```

Similar to the previous script, you declare a `URL` variable containing the web URL that you want to scrape and a `videos` variable that stores all the data as a list. 

Next, a `driver` variable is declared (_i.e._, a `Chrome` instance) that you use to interact with the browser. The `get()` function opens the browser instance and sends a request to the specified `URL`. 

After that, you call the `sleep` function to wait for three seconds before accessing any element on the web page to ensure all the HTML code is loaded in the browser.

Now, emulate scrolling down to load more videos:

```python
parent = driver.find_element(By.TAG_NAME, 'html')
for i in range(4):
    parent.send_keys(Keys.END)
    sleep(3)
```

The `send_keys` method simulates pressing the `END` key to scroll to the bottom of the page, triggering more videos to load. This action is repeated four times within a `for` loop to ensure enough videos are loaded. The `sleep` function pauses for three seconds after each scroll to allow the videos to load before scrolling again.

Next, extract video titles and links:

```python
html_data = html.fromstring(driver.page_source)

videos_html = html_data.cssselect("a#video-title-link")
for video in videos_html:
    title = video.text_content()
    link = "https://www.youtube.com" + video.get("href")

    videos.append( {"title": title, "link": link} )
```

In this code, you pass the HTML content from the driver’s `page_source` attribute to the `fromstring` method, which builds a hierarchical tree of the HTML. 

Then, you select all `<a>` tags with the ID `video-title-link` using CSS selectors, where the `#` sign indicates selection using the tag’s ID. This selection returns a list of elements that satisfy the given criteria. 

The code then iterates over each element to extract the title and link. The `text_content` method retrieves the inner text (the video title), while the `get` method fetches the `href` attribute value (the video link). 

Finally, the data is stored in a list called `videos`.

Now, save the data as a JSON file and close the driver:

```python
with open('videos.json', 'w') as file:
    json.dump(videos, file)
driver.close()
```

Run the script:

```sh
python dynamic_scrape.py
```

After running the script, a new file named `videos.json` is created in your directory:

![dynamic_scrape.py JSON output](https://brightdata.com/wp-content/uploads/2024/08/dynamic_scrape.py-JSON-output-1024x495.png)

All the code for this script is also available on [GitHub](https://gist.github.com/vivekthedev/36489fbaf896eb7c06ebb9350dec298a).

### Using lxml with Bright Data Proxy

Web scraping can face challenges like anti-scraping tools and rate limits. Proxy servers help by masking the user’s IP address. Bright Data provides reliable proxy services.

To start, obtain proxies from Bright Data by signing up for a free trial. After creating a Bright Data account, you’ll see the following dashboard:

![Bright Data Dashboard](https://brightdata.com/wp-content/uploads/2024/08/Bright-Data-Dashboard-1024x461.png)

Navigate to the **My Zones** option and create a new residential proxy. This will reveal your proxy username, password, and host, which you need in the next step.

Next, modify `static_scrape.py` by adding the following code below the URL variable:

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

Replace placeholders with your Bright Data credentials and run the script:

```sh
python static_scrape.py
```

After running this script, you’ll see a similar output to what you received in the previous example.

You can view this entire script on [GitHub](https://gist.github.com/vivekthedev/201f994bc14e4dbc7263b03983f917b3).

## Conclusion

Using lxml with Python enables efficient web scraping, but it can be time-consuming. Bright Data offers an efficient alternative with its ready-to-use [datasets](https://brightdata.com/products/datasets) and [Web Scraper API](https://brightdata.com/products/web-scraper).

Try Bright Data for free!