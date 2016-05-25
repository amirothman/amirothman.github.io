---
layout: post
title: Building A News Crawler
---

One of the things that I enjoy doing is text mining. Basically going through huge
text corpora like a mad man searching for different patterns. In order to feed this odd habit,
I need collections of texts. There are countless public corpora available, but
building your own from scratch rewards some kind of an unexplainable joy. It might be one
of those tasks which are not too hard to be overwhelming, yet challenging enough
to be rewarding. I might be broken. I might need help. But that's a different story.

I used to do this with Ruby utilizing [Nokogiri](http://www.nokogiri.org/). But I found out that Python has so much more tools for automating almost everything, so in this post I'm going to describe the steps that I took to build this crawler. I realize I might be using the term crawler too liberally here, but my dumb feelings are telling me the word scraper is less accurate. Scream in the comment section if you please. And some of the tools are completely new to me, so forgive me if I mess up some of the steps.

So, let's get it going. First, we need to identify our source for text. Lately, I enjoy
mining news articles. And on top of that, you can almost be certain that the patterns
that you uncover are new patterns that no one else has discovered. Cool, we have a place to start.

The best place to get news programmatically might be RSS Feeds. RSS Feeds might not be
as hip as they once were, but most news portals still provide them. How to read these feeds?

I found [feedparser](https://pythonhosted.org/feedparser/introduction.html#parsing-a-feed-from-a-remote-url). This is pretty sweet. So install it on Python.

    pip install feedparser

Their documentation is awesome. I got my feedparser up and running in no time. To test out my parser I will be using [BorneoPost](http://www.theborneopost.com/feed/) as a news feed. I played around in IPython to get some familiarity.

    import feedparser
    parsed = feedparser.parse("http://www.theborneopost.com/feed/")
    parsed.entries

Nice. Now I can create a new list of links.

    links = [el["link"] for el in parsed.entries]

Now I need to extract the content from each link. Content extraction refers to extracting the main content from the sites. For us humans, this task is trivial. But for computers, the whole site is a HTML document. We could write a program that parses the HTML and extracts the DOM by telling the program where the main text is but this would also mean we would have to write a different program when we want to extract contents on a different site, which may have different formatting. Hence, we need the help of content extractors. They usually utilize some clever machine learning models to select which DOM most probably holds the main content. If you are interested you can [read here](http://www2013.org/companion/p89.pdf) to get some idea.

I am aware of [dragnet](https://pypi.python.org/pypi/dragnet), so I tried to install it with pip, but received some error. Then I realized, it does not support Python 3. Then I read one of the [blog posts](https://moz.com/devblog/benchmarking-python-content-extraction-algorithms-dragnet-readability-goose-and-eatiht/) from the developers and found out about other popular content extractors.
So I thought to myself, I might as well just try one of the others.

At random, I chose [python-readibility](https://pypi.python.org/pypi/readability-lxml). Installed it with:

    pip install readability-lxml

Installation worked like a charm. So, I shall be sticking with this one if the results from the content extraction is reasonable and acceptable. Let's try it out.

    from readability.readability import Document
    import requests
    r  = requests.get(links[0])
    html = r.text
    readable_article = Document(html).summary()

Ok, readable_article, filtered out most of the unwanted elements. But there are still some HTML tags. I don't want that. I tried feeding ``readable_article`` back to ``Document().summary()``, but I get the same result.

I remembered using [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) with its ``.text`` method to get rid of HTML tags. I think it was from some text mining tutorial I can't quite recall. Anyways, let's try that out and see if it works.

    from bs4 import BeautifulSoup
    cleaned_readable_article = BeautifulSoup(readable_article).text
    # 'KUALA LUMPUR: 1Malaysia Development Bhd (1MDB...'

I've truncated the output a bit, but in my eyes it works. Now, we have all the tools to start building our corpus. A recap to what we will be doing.

  * We need a collection of RSS Feeds
  * From the feeds, we get the links and published date of each article
  * From the links, we extract the main text
  * We shall then save each text in a JSON file

Now all we need is to bring everything together. My example can be accessed [here](https://gist.github.com/amirothman/e29875f0f34148ecf340d5039664ccd2). If you utilize a while loop with a sleep, you can have yourself a nice simple crawler. Collect more news feeds and let your crawler run for some time. Watch it grow like a little baby. Be aware however, certain sites do not like the fact that you are scraping. Be polite to not DDOS their servers. We all gotta eat.

On the next post I will demonstrate some ways we can augment and explore our text data to uncover some cool patterns.
