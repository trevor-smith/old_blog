---
layout: post
title: Scraping the pain away
---

This last week was all about scraping data.  I had always wanted to learn this skill
so I was keen to get started!  But where to begin for a noob like myself??

Luckily, as usual python has an easy to implement module that will have you scraping
data in a few minutes and it's called BeautifulSoup.  Seriously, you can just read
their documentation, maybe follow a tutorial or two, and start scraping away!  And this
is coming from someone who's never even touched html before.

For our project Luther, our data source to scrape was a movie website called
BoxOfficeMojo.  It has A TON of movie related data for movies spanning back to the early
90's.  The goal of this project was to scrape data from BoxOfficeMojo and use it
to answer an interesting business problem.  This post isn't going to get into my problem 
I was solving (that's the next post), but I do want to share some of my code.

# Let's get scraping!

First, you need to import the following packages (and install them too if you don't have):

````python
import urllib2
from bs4 import BeautifulSoup
import re
````

Next, I chose to scrape all the american movies on the website, but to do that
I would need to get every url of every american movie.  To do this I located
a webpage that had all movies listed.  There were 135 pages to go through so I quickly
found the url structure and wrote some code to loop through them all

````python
# getting the list of all the domestic urls to scrape
# this will help get us the list of movies and their respective id's
# with these page id's we can construct the url list to scrape
url_domestic_list = []
page_number = range(0,135)
for i in page_number:
    url_domestic_list.append("http://www.boxofficemojo.com/alltime/domestic.htm?page=" 
    + str(page_number[i]) + "&p=.htm")'
````  

Now that we had the url list to go through, it was time to go through each page and extract
the movie ids.  With these movie ids I could then construct the url of the movie (thanks
BoxOfficeMojo for having nice pagination :D).  You'll see below that I'm keying off
of 'a' tag and looking for the href that begins with '/movies/'.  There's certainly
other way's to do this sort of thing and I encourage you to explore various ways
of scraping this data!  

````python
# looping through all the domestic url pages and scraping the movie urls
id_list = []
movie_url_to_scrape = []
for url in url_domestic_list:
    page_domestic = urllib2.urlopen(url)
    soup_domestic = BeautifulSoup(page_domestic)
    # need to encode this as utf-8 or else it throws error
    # for the a attribute, look for hrefs that start with '/movies/?'
    href_movie_tags = [(a.attrs.get('href')).encode('utf-8') for a in soup_domestic.select('a[href^/movies/?]')]
    # splitting the hrefs based on movie id
    href_movie_tags_split = [i.split('id', 1) for i in href_movie_tags]
    # create a list of only the movie ids
    for i in href_movie_tags_split[:-1]:
        id_list.append(i[1])
    id_list_unique = set(id_list)
	# creating the list of movie urls that we will be scraping
	# this is done by using a templated url and adding the id to make a valid movie url
    for id in id_list:
        movie_url_to_scrape.append("http://www.boxofficemojo.com/movies/?id" + id)
    movie_url_to_scrape_unique = set(movie_url_to_scrape)
````

# We have movie url's, NOW WHAT?

The last part I'm going to discuss is how to get the data we want from each webpage.
This part wasn't as straightforward as the above mainly because we wanted to scrape
many different items at different positions on the page and the code I wrote above
was not going to do the trick.

I did notice some patterns though and realized that I could just key in the label
name of what I was looking for.  For example, if I wanted to scrape 'Domestic Total Gross'
then there was an html attribute labeled 'Domestic Total Gross'.  Once I located that,
I then just had to extract the value underneath it.  The function is below:

````python
def get_movie_value(soup, field_name):
    """
    takes a string attribute of a movie on the page, and
    returns the string in the next sibling object (the value for that attribute)
    """
    obj = soup.find(text = re.compile(field_name))
    if not obj:
        return None
    next_sibling = obj.findNextSibling()
    if next_sibling:
        return next_sibling.text
    else:
        return None
````

And there you have it!  A robust function to scrape BoxOfficeMojo.  Next week, I'll
show you what data I pulled and how I analyzed it.  Until then...stay classy.

~ Trevor