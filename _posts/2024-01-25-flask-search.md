---
title: "Simple Flask Integration for an Elastic Semantic Search App"
date: 2024-01-25
---

### Have you been on any websites without a search function lately? 
How about ones that didn't allow for typos, synonyms, or "I forgot the exact word but I'm looking for something to do with these themes?". Probably not... or if you did it was likely a frustrating experience. 

Semantic (or vector) Search is becoming ubiquitous in the way we interact with the internet - we're always looking for something, and we want to find it no matter how bad we may be at formulating what it is!

### Enter Elasticsearch.
 
With the Lucene-based engine you can quickly build an [index](https://www.elastic.co/guide/en/elasticsearch/reference/current/documents-indices.html) of documents (be it numbers, words, images, or sound) and start searching through them based on various filters, aggregations, and fancy new-age models like [ELSER](https://www.elastic.co/guide/en/machine-learning/current/ml-nlp-elser.html). 

If this is your first time using Elastic, you can check out my full tutorial [starting-point repo](https://github.com/iuliaferoli/harry-potter-search) here or find some cool videos on the [Community YouTube Channel](https://www.youtube.com/@OfficialElasticCommunity). 

In this blog, we're going to address the "on any website" part of a Search Solution. Or at least - propose a starting point for it. There are many great tutorials out there for a deep dive on Flask - one of the best [from my colleague Miguel](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world). 

However, as someone getting reacquainted/jumping deeper into the dev realm after mostly shallow toe-dipping I found some of the guides a bit overwhelming. So this will be a very simple, stripped-down, essentials-only example of a basic Flask app. 

### So what's our minimally viable "Website with search"?
We have a Search Engine we've built for our custom domain. (You can think of a retail online store, blogging website, cooking recipe inventory, news website, etc - what I'm using is the Harry Potter books, the content is mostly irrelevant.)

On the Elastic Side, you can see how I built my Semantic [Search App here](https://github.com/iuliaferoli/harry-potter-search?tab=readme-ov-file#harry-potter-movie-dialoogue-index--intro-to-elasticsearch-python-client). 

What we want now is to abstract all that and only use a few simple functions to call the functionalities we need. Namely, connect to the client; run the actual user input as a semantic search on our index, and log the search to our history. See [these functions here](https://github.com/iuliaferoli/harry-potter-search/blob/main/helper_functions.py).

Now let's Flask!
I'm building three pages - so we will an HTML template file, and a function and API mapping for each.
Our folder structure:

<img src="/_posts/img/folder_structure.png" width="300">

See the full [Python web_app](https://github.com/iuliaferoli/harry-potter-search/blob/main/web_app.py) file here
See the full [HTML template files](https://github.com/iuliaferoli/harry-potter-search/tree/main/templates) here

The website pages will end us looking like this:
<img src="/_posts/img/web_app.jpeg" width="500">


### 1. User input for our search.

The premise is simple - one simple form and one button. 

Create the routes in the file we define the Flask ```app = Flask(__name__)```

```python
@app.route('/')
def search():
   return render_template('search.html')
```

And create a simple HTML template:

```html
{% raw %}
<form action = "http://127.0.0.1:5000/search" method = "POST">
    <p>Your Query: <input type = "text" name = "question" /></p>
    <p><input type = "submit" value = "submit" /></p>
 </form>
{% endraw %}
```

A user can now type in their query; and behind the scenes it is run in Elasticsearch, and the user is routed to a page where they can see the results, namely: 

### 2. View result of search

This is the meat and potatoes of our project. For the user, it's another simple static page; but routing to this page runs the helper functions we mentioned earlier - interacting with our Elastic backend. 
On the surface though, we quickly get the top results for our search in a few seconds. Users want to find stuff fast! 


```python
@app.route('/search' ,methods = ['POST', 'GET'])
def show_search_term():
    if request.method == 'POST':
        # getting the query from the user
        question = request.form["question"]
        # running the semantic search model and getting the results from Elasticsearch
        answer = semantic_search(question, client=client,  model_id=model_id, index=index)
       
        # Logging the search & response in a separate index
        document = {"Query" : question, "Response" : answer, "date" : datetime.now()}
        response = client.index(index = "historical_searches", document = document)
        #print(response)

        # Returning the template for the user to view their results
        return render_template('search_result.html', answer=answer, question =question)
```

This is rendered with a jinja2 template (allowing the HTML to use for statements to iterate through lists) allowing us to loop through the list of documents we get back from Elastic and show them on the page:

```html
<body>
<h1>Your query was: {{ question }}</h1>
<p>Search Results:</p>
<ul id="answer">
{% for item in answer %}
<li> {{ item }} </li>
{% endfor %}
</ul>
</body>
```

Lastly, if you want to see previous searches, you can go to:

### 3. History of Searches.

This page triggers a separate call to Elastic, asking to give us back the latest queries and answers that have been run by users. If the previous was meat and potatoes, this is... gravy?

```python
@app.route('/history')
def show_history():
   response = client.search(index = "historical_searches", sort=[{"date" : {"order": "desc"}}])
   return render_template('history.html', response = response["hits"]["hits"])
```

The template is very similar to the previous, just with one extra for loop to show multiple lists on answers:

```html
<h1>These are the past searches ran:</h1>
    
{% for result in response %}
<p>Your query was: {{ result._source.Query }}</p>
<p>Search Results:</p>
<ul id="answer">
{% for item in result._source.Response %}
<li> {{ item }} </li>
{% endfor %}
</ul>

{% endfor %}
```

There we go!

With a few simple lines of code, you can turn a search engine into a "Website with a Search Function". Naturally, you will more likely include these capabilities in your existing web infrastructure - which is surely more sophisticated than three Flask pages. 

Hopefully, this blog mostly works to illustrate the user experience - and how close to your fingertips it is!

Happy Searching! 
Next up - making these pages not look horrible and expanding on the search web app idea. Stay tuned :) 





