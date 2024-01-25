<body>
    
    <h2>Welcome to Iulia's blog</h2>
    <br>
    <p>I'm Iulia (iulia not Lulia) and I work in tech, more specifically as a Developer Advocate (aka devrel//tech evangelism/other buzzwords). Currently, I'm @ Elastic.
    <br>
    Basically, I build content & demos with Python, mostly about ML, NLP, LLMs, and semantic search/embeddings/vectors. And you know, search.
    <br>
    You can find me at tech conferences across EMEA (mostly Norther Europe) either at one of our Elastic booths or speaking about Harry Potter for some reason.
    I also host the meetups at our Elastic Amsterdam office (one every month, we have pizza!)
    </p>
    
    <ul>
    {% for post in site.posts %}
        <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
        </li>
    {% endfor %}
    </ul>


</body>
