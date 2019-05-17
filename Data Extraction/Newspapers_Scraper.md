# Data Extraction methods used for Regional Newspapers

## Targeted Newspapers Regions are

- Afghanistan
- Bangladesh
- Indonesia
- India
- Vietnam

### Data Extraction parameters as follows
- Headline
- Description
- Author
- Published Date
- Publication
- Category
- News
- URL
- Keywords
- Summary

## Getting Started

### Python Dependencies

```bash
$ pip install selenium
$ pip install beautifulsoup4
$ pip install newspaper3k
$ pip install pandas
```

### Code has been divided into Three section

  * URL Extraction
  * Article details Extraction
  * Creating Data Frame

***First part of the code is mostly on URL's extraction follows similar method with help of [BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) & [Selenium](https://selenium-python.readthedocs.io/)***

- Google Custom Keyword Searching

```bash
'HIV site: "name of the website"'
```

- Finding total number of Google Search Results

```python3
keyword = 'HIV site:www.dailysun.co.za'

url = 'https://www.google.com/search?q=' + '+'.join(keyword.split())

soup = BeautifulSoup(get(url).text, 'lxml')
try:
    # Extracts the digits if it the resulted number without comma ','. eg: About 680 results (0.23 seconds)
    max_pages = round([int(s) for s in soup.select_one('div#resultStats').text.split() if s.isdigit()][0]/10)
    max_pages = max_pages + 1
except:
    # Extracts the digits if it the resulted number without comma ','. eg: About 1,080 results (0.23 seconds)
    max_pages = round(int(''.join(i for i in soup.select_one('div#resultStats').text if i.isdigit()))/10)
    max_pages = max_pages + 1
```

- Scraping the resulted urls by iterating the max_pages through while loop  

```python3
options = Options()
options.headless = True
browser = webdriver.Chrome(options=options)
browser.get(url)

index = 0

while True:
    try:
        index +=1
        page = browser.page_source
        soup = BeautifulSoup(page, 'lxml')
        linky = [soup.select('.r')[i].a['href'] for i in range(len(soup.select('.r')))]
        urls.extend(linky)
        if index == max_pages:
            break
        browser.find_element_by_xpath('//*[@id="pnnext"]/span[2]').click()
        time.sleep(2)
        sys.stdout.write('\r' + str(index) + ' : ' + str(max_pages) + '\r')
        sys.stdout.flush()
    except:
        pass

browser.quit()
```

***Second part of the code is mostly on extraction of News Articles details & basic NLP stuff regards Keywords in the articles and Summary of the article with the help of [NewsPaper3K](https://pypi.org/project/newspaper3k/)***

- Scraping the news article details by iterating the urls through the for loop

```python3
for index, url in enumerate(urls):
    try:
        # Parse the url to NewsPlease 
        article = Article(url)
        article.download()
        article.parse()
        article.nlp()

        # Extracts the Headlines
        try:
            headlines.append(article.title.strip())
        except:
            headlines.append(None)

        # Extracts the Descriptions    
        try:
            descriptions.append(article.meta_description.strip())
        except:
            descriptions.append(None)

        # Extracts the Authors
        try:
            authors.append(article.authors.strip())
        except:
            authors.append(None)

        # Extracts the published dates
        try:
            dates.append(article.meta_data['article']['published_time'].split('T')[0])
        except:
            dates.append(None)

        # Extracts the news category
        try:
            category.append(article.meta_data['category'])
        except:
            category.append(None)

        # Extracts the news articles
        try:
            news.append(' '.join(article.text.split()).replace("\'\'"," ").replace("\'", "").replace(" / ", ""))
        except:
            news.append(None)

        # Extracts the news publication
        try:
            publication.append(article.meta_data['og']['site_name'])
        except:
            publication.append(None)

        # Extracts Keywords and Summaries
        try:
            keywords.append(article.keywords)
            summaries.append(' '.join(article.summary.split()))
        except:
            keywords.append(None)
            summaries.append(None)

    except:
        headlines.append(None)
        descriptions.append(None)
        authors.append(None)
        dates.append(None)
        category.append(None)
        publication.append(None)
        news.append(None)
        keywords.append(None)
        summaries.append(None)

    sys.stdout.write('\r' + str(index) + ' : ' + str(url) + '\r')
    sys.stdout.flush()
```

***Final Step***

- From the collected data creating the [Pandas](https://pypi.org/project/pandas/) DataFrame and droping the Nan values

```python3
if len(headlines) == len(descriptions) == len(authors) == len(dates) == len(news) == len(publication) == len(keywords) == len(summaries) == len(urls) == len(category):
  tbl = pd.DataFrame({'Headlines' : headlines,
                      'Descriptions' : descriptions,
                      'Authors' : authors,
                      'Published_Dates' : dates,
                      'Publication' : publication,
                      'Articles' : news,
                      'category' : category,
                      'Keywords' : keywords,
                      'Summaries' : summaries,
                      'Source_URLs' : urls})
  tbl.dropna()
  print('Enter the path to store the file')
  path = input()
  tbl.to_csv(path+'Daily_Star.csv', index=False)
else:
   print('Array lenght does not match!')

tbl.head()
```
