# Scraping & API Cookbook
## I. HTTP
### HTTP request structure

* Request Line
* Header Field
* Body

### Make HTTP request using curl (command line tool)
```
curl -v https://arxiv.org/ 
# -v means verbose

curl -v https://arxiv.org/search/?query=CVPR+2023+NeRF&searchtype=all&source=header
# a request with query string included
```

### Make HTTP request using requests (a Python module)

#### GET request

requests.get(url, params=None, **kwargs)
```
import requests
res = requests.get('https://bing.com')
text = res.text
```
* return a `Response` object 
* `res.text` returns a string that containing the entire response
* response body is often json(API) or HTML(access a web page)
* request data from a server

#### POST request

requests.post(url, data=None, json=None, **kwargs)
```
res = requests.post('https://httpbin.org/post',
                     data={'name': 'King Triton'})
```
* return a `Response` object 
* `res.text` returns a string that containing the entire response
* response body is usually json
* send data to a server

#### HTTP status codes
```
code = res.status_code
```

* return HTTP status code of the response

```
ok = res.ok
```

* return a boolean representing whether the request is success or not

|code|representing|
|-|-|
|200|no issues|
|400|bad request|
|404|page not found|
|405|method not allowed|
|500|internal server error|

|code class|representing|
|-|-|
|100 – 199|Informational responses| 
|200 – 299|Successful responses| 
|300 – 399|Redirection messages| 
|400 – 499|Client error responses| 
|500 – 599|Server error responses| 

## II. JSON
* json.load()
  ```
  with open('text.json','r',encoding='utf-8') as f :

  json.load(f)
  ```
  Read json as dict from file.

* json.loads()
  ```
  L = [json.loads(x) for x in open(os.path.join('data', 'quotes2scrape.json'))]
  ```
  Read str as dict in json format. Dangerous when the string is not a typical json format. For instance, `util.err()` with no quotes around it will cause error since it is not a string in json. Json won't accept such format. 

* json.dumps()

  Read dict as str.

* json.dump()

  Write into json.

* dict.keys()
  ```
  js_dict.keys()
  js_dict["key_name"].keys()
  ```
  Find the keys under this level of json.

* One-Hot Encoding
  ```
  def flatten_tags(taglist):
    return pd.Series({k:1 for k in taglist}, dtype=float)

  tags = df['tags'].apply(flatten_tags).fillna(0).astype(int)
  
  df_full = pd.concat([df, tags], axis=1).drop(columns='tags')
  ```
  

## III. HTML

### HTML anatomy


* HTML document: The totality of markup that makes up a webpage.

* Document Object Model (DOM): The internal representation of a HTML document as a hierarchical tree structure.

* HTML element: An object in the DOM, such as a paragraph, header, or title.

* HTML tags: Markers that denote the start and end of an element, such as \<p> and \</p>.

### HTML tags

|Element|Description|Attributes|
|---|---|---|
|`<html>`|the document||
|`<head>`|the header||
|`<body>`|the body||
|`<div>` |a logical division of the document|all kinds of attributes|
|`<span>`|an *inline* logical division||
|`<p>`|a paragraph||
| `<a>`| an anchor (hyperlink)|href:url link|
|`<h1>, <h2>, ...`| header(s) ||
|`<img>`| an image |src:file path, alt:description(will be displayed if the image isn't successfully loaded)|
|`<ul>`|define orderless list||
|`<li>`|define an object in the list, often appears with `<ul>`||

* \<div> tag defines a division or a "section" of an HTML document. The \<div> element is often used as a container for other HTML elements to style them with CSS or to perform operations involving them using JavaScript. \<div> elements often have attributes, which are important when scraping!

### HTML structure
```
<html> -><head> -><title>
       -><body> -><div> -><div>
                        -><div>
                        -><div> -><p>
                                -><h3>
                                -><ul>                        
                -><div> -><ul> -><li>
                               -><li>
                               -><li>
```

### Parsing HTML with Beautiful Soup 4 (a Python module)

#### Parsing Functions
* bs4.BeautifulSoup(text)
  ```
  import bs4
  html_string = requests.get('https://www.arxiv.org').text
  soup = bs4.BeautifulSoup(html_string)
  ```
  Input a raw HTML text in string type, return bs4.BeautifulSoup.

* soup.text

  return the text part of the soup

* soup.descendants
  ```
  list(soup.descendants)
  
  for child in soup.descendants:
      print(child)                 # return the child
      print(child.name)            # return its name (the outer tag)
  ```
  The descendants attribute traverses a BeautifulSoup tree using depth-first traversal.

* soup.find(name=None, attrs={}, recursive=True, text=None, **kwargs)
  ```
  div = soup.find('div') # find the first <div>
  div = soup.find('div', attrs={'id':'nav'}) * find the first <div> with attribute id = 'nav'
  ```
  Finds the first instance of a tag (the first one on the page, i.e. the first one that DFS sees. Find will return the first occurrence of a tag, regardless of its depth in the tree.

* soup.find_all(name, attrs, recursive, string, **kwargs)
  ```
  divs = soup.find_all('div')
  ```
  return a list containing all the matches in the soup

#### Node Attributes

|Attribute|Function|
|--|--| 
|node.text|return the text part between the tags|
|node.attrs|return all the attributes of a tag in a dict|
|node.get(key)|get the value of a tag attribute|

```
soup.find('p').text
soup.find('div').attrs
soup.find('div').get('id')
```
* Use .text.strip() to get a cleaner text!

## IV. API
```
r= requests.get(url)
r_js = r.json()
```


## V. Scraping

### Scraping Example

```
fac_response = requests.get('https://global.sjtu.edu.cn/en/cooperation/globalclass')
fac_response

soup = bs4.BeautifulSoup(fac_response.text)

divs = soup.find_all('div', attrs={'class': 'layui-col-lg3 layui-col-md4 layui-col-sm6 layui-col-xsm6 layui-col-xs12'})
divs[0].find('div',attrs={'class':'large-title'}).text.strip()
divs[0].find('a').get('href')
divs[0].find_all('p')[1].text
divs[0].find_all('p')[0].text

names = [div.find('div',attrs={'class':'large-title'}).text.strip() for div in divs]
names[:5]

credits = [div.find_all('p')[1].text for div in divs]
credits[:5]

departments = [div.find_all('p')[0].text for div in divs]
departments[:5]

links = [ 'https://global.sjtu.edu.cn'+div.find('a').get('href') for div in divs]
links[:5]

courses = pd.DataFrame().assign(name=names, credit=credits, department=departments ,link=links)
courses.head()
```

### Getting a Web page
```
def download_page(i):
    url = f'https://quotes.toscrape.com/page/{i}'
    request = requests.get(url)
    return bs4.BeautifulSoup(request.text)
```

### Parsing a single page
```
def process_quote(div):
    quote = div.find('span', attrs={'class': 'text'}).text
    author = div.find('small', attrs={'class': 'author'}).text
    author_url = 'https://quotes.toscrape.com' + div.find('a').get('href')
    tags = div.find('meta', attrs={'class': 'keywords'}).get('content')
    
    return pd.Series({'quote': quote, 'author': author, 'author_url': author_url, 'tags': tags})
```

### Merge all the parsings
```
def process_page(divs):
    return pd.DataFrame([process_quote(div) for div in divs])
```

### Combination
```
def quote_df(n):
    '''Returns a DataFrame containing the quotes on the first n pages of https://quotes.toscrape.com/.'''
    dfs = []
    for i in range(1, n + 1):
        # Download page n and create a BeautifulSoup object.
        soup = download_page(i)
        
        # Create DataFrame using the information in that page.
        divs = soup.find_all('div', attrs={'class': 'quote'})
        df = process_page(divs)
        
        # Append DataFrame to dfs.
        dfs.append(df)
        
    # Stitch all DataFrames together.
    return pd.concat(dfs).reset_index(drop=True)
```




