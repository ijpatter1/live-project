# The Tech Academy Live Project

Welcome to my Live Project repository. This project was built using the Django framework. I was tasked with building an interactive web app for managing one's
collections of things related to various hobbies, as well as API and Data Scraped content for those hobbies.

There were 3 distinct portions of the app with individual stories covering each portion: a Database Collection Manager, Data Scraping with Beautiful Soup and a Restful API interface.

The tool set for the project consisted of the following Python packages: beautifulsoup4 4.7.1; certifi 2018.11.29; chardet 3.0.4; Django 2.1.5; idna 2.8; numpy 1.16.2; pytz 2018.9; requests 2.21.0; selenium 3.141.0; soupsieve 1.7.2; urllib3 1.24.1.

# Database collection manager

INSERT GIF HERE

Specifically, my web app focused on tracking S&P 500 quarterly earnings report data. For context, the S&P 500 is an index of 500 large companies list on US stock exchanges.
I created a django model and model form for saving company data to the database and developed basic "CRUD" functionality. This included a details page that featured widgets providing more in depth data.

# Restful API interface
A Restful API is an internet service that returns usually JSON responses based on url/http queries. This
is a commonly used element in web services.
You'll add in a page for information from an API related in some way to the item your collection..
When picking an API you are looking for something that is:
Free to use
Allows non-commercial apps to integrate it (or doesn't require you to register your app at all)
Has no authorization or uses an APIKey (OAuth is too complicated)
Has good documentation for how to utilize it
You are asked to pick an API at the beginning so it can be checked for these elements. You can
change it later if needed.
# Data Scraping with Beautiful Soup
Beautiful Soup and Data Scraping are methods of gathering information from existing web pages.
You'll be pulling information from a section of a web page to integrate into your app. Again, this can
be a related page.
When picking a page to scrape you are looking for:
A page that uses ids and/or classes within the source
A page that has a unique class or id for the section(s) you want to pull information from
7/6/2020 Creating Your App - Overview
https://prosperitprojects.visualstudio.com/AppBuilder9000/_wiki/wikis/AppBuilder9000.wiki/137/Creating-Your-App 2/4
Preferably a page with changing content so using scraping makes sense (this is more for the
portfolio aspect of it)
Certain scripting or other blockers can prevent data scraping ability. So try to have an idea of a
backup page should you need to find a new one to work with.
