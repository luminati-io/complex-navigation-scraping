# Scraping Websites With Complex Navigation

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to use Selenium and browser automation to scrape websites with complex navigation patterns, such as dynamic pagination, infinite scrolling, and ‘Load More’ buttons, using Selenium and browser automation.

- [What Is Considered Complex Navigation?](#what-is-considered-complex-navigation)
- [Tools to Handle Complex Navigation Websites](#tools-to-handle-complex-navigation-websites)
- [Scraping Common Complex Navigation Patterns](#scraping-common-complex-navigation-patterns)
  - [Dynamic Pagination](#dynamic-pagination)
  - [‘Load More’ Button](#load-more-button)
  - [Infinite Scrolling](#infinite-scrolling)
- [Conclusion](#conclusion)

## What Is Considered Complex Navigation?

In web scraping, complex navigation refers to website structures where content or pages are not easily accessible. Complex navigation scenarios often involve dynamic elements, asynchronous data loading, or user-driven interactions. These aspects may enhance user experiences, but they also significantly complicate data extraction. Here are some very common examples:

- **JavaScript-rendered navigation**: Websites that rely on JavaScript frameworks to generate content directly in the browser.
- **Paginated content**: Sites with data spread across multiple pages where pagination is loaded dynamically via AJAX.
- **Infinite scrolling**: Pages that load additional content dynamically as users scroll, typical for social media feeds, Discourse-based forums, and news websites.
- **Multi-level menus**: Sites with nested menus requiring multiple clicks or hover actions to reveal deeper layers of navigation, common for product category trees on marketplaces.
- **Interactive map interfaces**: Websites displaying data on maps or graphs, where information is dynamically loaded as users pan or zoom.
- **Tabs or accordions**: Pages with content hidden under dynamically rendered tabs or collapsible accordions that are not directly embedded in the HTML returned by the server.
- **Dynamic filters and sorting options**: Sites with complex filtering systems where applying multiple filters reloads the item listing dynamically without altering the URL structure.

## Tools to Handle Complex Navigation Websites

Many of the complex interactions listed above need JavaScript execution, something only a browser can do. This means you cannot rely on simple [HTML parsers](https://brightdata.com/blog/web-data/best-html-parsers) for such pages. Instead, you must use a browser automation tool like Selenium, Playwright, or Puppeteer. These solutions allow you to programmatically instruct a browser to perform specific actions on a web page, mimicking user behavior.

## Scraping Common Complex Navigation Patterns

This guides covers three specific types of complex navigation patterns:

- **Dynamic pagination**: Sites with paginated data loaded dynamically via AJAX.
- **‘Load More’ button**: A common JavaScript-based navigation example.
- **Infinite scrolling**: A page that continuously loads data as the user scrolls down.

We will use Selenium in Python, but the logic can be adapted to Playwright, Puppeteer, or any other browser automation tools. The guide also assumes that you are already familiar with the basics of [web scraping using Selenium](https://brightdata.com/blog/how-tos/using-selenium-for-web-scraping).

### Dynamic Pagination

We will use the “[Oscar Winning Films: AJAX and Javascript](https://www.scrapethissite.com/pages/ajax-javascript/#2014)” scraping sandbox:

![The target page. Note how pagination data is loaded dynamically](https://github.com/luminati-io/complex-navigation-scraping/blob/main/Images/Dynamic-pagniation-example-1536x752.gif)

This site dynamically loads Oscar-winning film data, paginated by year.

To navigate and scrape such a page effectively, you need to follow the following steps:

1. Click on a new year to trigger data loading (a loader element will appear).
2. Wait for the loader element to disappear (indicating the data has fully loaded).
3. Verify that the table with the data has been properly rendered on the page.
4. Scrape the data once it becomes available.

Below is an example of how to implement this logic using Selenium in Python:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options

# Set up Chrome options for headless mode
options = Options()
options.add_argument("--headless")

# Create a Chrome web driver instance
driver = webdriver.Chrome(service=Service(), options=options)

# Connect to the target page
driver.get("https://www.scrapethissite.com/pages/ajax-javascript/")

# Click the "2012" pagination button
element = driver.find_element(By.ID, "2012")
element.click()

# Wait until the loader is no longer visible
WebDriverWait(driver, 10).until(
    lambda d: d.find_element(By.CSS_SELECTOR, "#loading").get_attribute("style") == "display: none;"
)

# Data should now be loaded...

# Wait for the table to be present on the page
WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.CSS_SELECTOR, ".table"))
)

# Where to store the scraped data
films = []

# Scrape data from the table
table_body = driver.find_element(By.CSS_SELECTOR, "#table-body")
rows = table_body.find_elements(By.CSS_SELECTOR, ".film")
for row in rows:
    title = row.find_element(By.CSS_SELECTOR, ".film-title").text
    nominations = row.find_element(By.CSS_SELECTOR, ".film-nominations").text
    awards = row.find_element(By.CSS_SELECTOR, ".film-awards").text
    best_picture_icon = row.find_element(By.CSS_SELECTOR, ".film-best-picture").find_elements(By.TAG_NAME, "i")
    best_picture = True if best_picture_icon else False

    # Store the scraped data
    films.append({
      "title": title,
      "nominations": nominations,
      "awards": awards,
      "best_picture": best_picture
    })

# Data export logic...

# Close the browser driver
driver.quit()
```

Here is the breakdown of that code snippet:

1.  The code sets up a headless Chrome instance.
2.  The script opens the target page and clicks the “2012” pagination button to trigger data loading.
3.  Selenium waits for the loader to disappear using [`WebDriverWait()`](https://selenium-python.readthedocs.io/waits.html).
4.  After the loader disappears, the script waits for the table to appear.
5.  After the data is fully loaded, the script extracts details such as film titles, nominations, awards, and whether the film won Best Picture. The extracted information is then stored in a list of dictionaries.

The result will be:

```json
[
  {
    "title": "Argo",
    "nominations": "7",
    "awards": "3",
    "best_picture": true
  },
  // ...
  {
    "title": "Curfew",
    "nominations": "1",
    "awards": "1",
    "best_picture": false
  }
]
```

Keep in mind that there isn’t always a single best approach to handling this navigation pattern. Alternative methods may be necessary depending on the page's behavior. Here are some examples:

*   Use `WebDriverWait()` in combination with expected conditions to wait for specific HTML elements to appear or disappear.
*   Monitor traffic for AJAX requests to detect when new content is fetched. This may involve using browser logging.
*   Identify the API request triggered by pagination and make direct requests to fetch the data programmatically (e.g., using the [`requests` library](https://brightdata.com/blog/web-data/python-requests-guide)).

### ‘Load More’ Button

To illustrate JavaScript-based complex navigation scenarios involving user interactions, let's use an example of a 'Load More' button. The concept is straightforward: a list of items is displayed, and clicking the button loads additional items.

This time, the target site will be the [‘Load More’ example](https://www.scrapingcourse.com/button-click) page from the Scraping Course:

![The ‘Load More’ target page in action](https://github.com/luminati-io/complex-navigation-scraping/blob/main/Images/Clicking-on-the-load-more-button-1536x752.gif)

To handle this complex navigation scraping pattern, follow these steps:

1.  Find the ‘Load More’ button and click it.
2.  Wait for the new elements to load onto the page.

Here is the code to use with Selenium:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait

# Set up Chrome options for headless mode
options = Options()
options.add_argument("--headless")

# Create a Chrome web driver instance
driver = webdriver.Chrome(options=options)

# Connect to the target page
driver.get("https://www.scrapingcourse.com/button-click")

# Collect the initial number of products
initial_product_count = len(driver.find_elements(By.CSS_SELECTOR, ".product-item"))

# Locate the "Load More" button and click it
load_more_button = driver.find_element(By.CSS_SELECTOR, "#load-more-btn")
load_more_button.click()

# Wait until the number of product items on the page has increased
WebDriverWait(driver, 10).until(lambda driver: len(driver.find_elements(By.CSS_SELECTOR, ".product-item")) > initial_product_count)

# Where to store the scraped data
products = []

# Scrape product details
product_elements = driver.find_elements(By.CSS_SELECTOR, ".product-item")
for product_element in product_elements:
    # Extract product details
    name = product_element.find_element(By.CSS_SELECTOR, ".product-name").text
    image = product_element.find_element(By.CSS_SELECTOR, ".product-image").get_attribute("src")
    price = product_element.find_element(By.CSS_SELECTOR, ".product-price").text
    url = product_element.find_element(By.CSS_SELECTOR, "a").get_attribute("href")

    # Store the scraped data
    products.append({
        "name": name,
        "image": image,
        "price": price,
        "url": url
    })

# Data export logic...

# Close the browser driver
driver.quit()
```

To handle the 'Load More' button navigation pattern, the script:

1.  Records the initial number of products on the page
2.  Clicks the “Load More” button
3.  Waits until the product count increases, confirming that new items have been added

This approach is both efficient and versatile, as it eliminates the need to know the exact number of elements to be loaded. However, alternative methods can also achieve similar results.

### Infinite Scrolling

Infinite scrolling is a popular interaction widely used on social media and e-commerce platforms to enhance user engagement. In this case, the target will be the same page as above but with [infinite scrolling instead of a ‘Load More’ button](https://www.scrapingcourse.com/infinite-scrolling):

![infinite scrolling instead of a 'Load More' button](https://github.com/luminati-io/complex-navigation-scraping/blob/main/Images/Infinite-scrolling-example-1024x501.gif)

Most browser automation tools do not provide a direct method for scrolling down or up a page, and Selenium is not an exception. Instead, you need to execute a JavaScript script on the page to perform the scrolling operation.

The solution is to write a custom JavaScript script that scrolls down:

1.  A specified number of times, or
2.  Until no more data is available to load.

> **Note**:\
> Each scroll loads new data and increments the number of elements on the page.

After that, you can scrape the newly loaded content.

Here is the code to use infinite scrolling in Selenium:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait

# Set up Chrome options for headless mode
options = Options()
# options.add_argument("--headless")

# Create a Chrome web driver instance
driver = webdriver.Chrome(options=options)

# Connect to the target page with infinite scrolling
driver.get("https://www.scrapingcourse.com/infinite-scrolling")

# Current page height
scroll_height = driver.execute_script("return document.body.scrollHeight")
# Number of products on the page
product_count = len(driver.find_elements(By.CSS_SELECTOR, ".product-item"))

# Max number of scrolls
max_scrolls = 10
scroll_count = 1

# Limit the number of scrolls to 10
while scroll_count < max_scrolls:
    # Scroll down
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

    # Wait until the number of product items on the page has increased
    WebDriverWait(driver, 10).until(lambda driver: len(driver.find_elements(By.CSS_SELECTOR, ".product-item")) > product_count)

    # Update the product count
    product_count = len(driver.find_elements(By.CSS_SELECTOR, ".product-item"))

    # Get the new page height
    new_scroll_height = driver.execute_script("return document.body.scrollHeight")

    # If no new content has been loaded
    if new_scroll_height == scroll_height:
        break

    # Update scroll height and increment scroll count
    scroll_height = new_scroll_height
    scroll_count += 1

# Scrape product details after infinite scrolling
products = []
product_elements = driver.find_elements(By.CSS_SELECTOR, ".product-item")
for product_element in product_elements:
    # Extract product details
    name = product_element.find_element(By.CSS_SELECTOR, ".product-name").text
    image = product_element.find_element(By.CSS_SELECTOR, ".product-image").get_attribute("src")
    price = product_element.find_element(By.CSS_SELECTOR, ".product-price").text
    url = product_element.find_element(By.CSS_SELECTOR, "a").get_attribute("href")

    # Store the scraped data
    products.append({
        "name": name,
        "image": image,
        "price": price,
        "url": url
    })

# Export to CSV/JSON...

# Close the browser driver
driver.quit() 
```

This script handles infinite scrolling by first identifying the current page height and product count. It limits the scrolling process to a maximum of 10 iterations. During each iteration, it:

1.  Scrolls down to the bottom
2.  Waits for the product count to increase (indicating new content has loaded)
3.  Compares the page height to detect whether further content is available

If the page height remains unchanged after a scroll, the loop terminates, signaling that there is no more data to load.

## Conclusion

Web scraping can be challenging when complex navigation patterns are involved, businesses can make it even more difficult by employing anti-scraping measures to block automated scripts. Browser automation tools, like Selenium, cannot bypass those restrictions.

The solution is to use a cloud-based browser like [Scraping Browser](https://brightdata.com/products/scraping-browser) which integrates with Playwright, Puppeteer, Selenium, and other tools, automatically rotating IPs with each request. It can manage browser fingerprinting, retries, CAPTCHA solving, and more. Say goodbye to getting blocked when navigating complex sites!
