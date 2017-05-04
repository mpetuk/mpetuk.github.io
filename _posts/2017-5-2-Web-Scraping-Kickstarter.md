---
layout: post
title: Web-Scraping Kickstarter
---
---
“Luther is a brilliant but emotionally impulsive detective who is tormented by the dark side of humanity while hunting down murderers."

---

I love it that Metis names our projects after characters of different drama series and that degree of drama rises, appropriately, each week. As such, we went from “Law and Order SVU” (week 1) to “Luther” (week 2)  to “The Wire” (week 3).

![Luther]({{ site.baseurl }}/images/Luther.jpg)

And this story is about Project Luther.  The task was to come up with a question that can be answered by means of linear regression algorithm. Sounds easy enough, right? But wait - there is more! The data for the project was to be web-scraped.

### Scraping the World Wide Web ### 

I decided that Kickstarter.com is a good candidate with goal and pledged amounts and other project characteristics available and that it would be interesting, indeed,  to try to predict project success by predicting its pledged amount.

Quick refresher: Kickstarter is a platform for creative projects to get crowd-funded. If project’s goal met within given time period, pledges become real funds.

Kickstarter’s web site is built using XML data, which makes it difficult to scrape, but in the end, Selenium did the magic.

Selenium is a suite of tools to automate web browsers across many platforms. It allows, among other things, access to web sites and interaction with their content from Python, for example. I mean, look at this thing going:

![demo]({{ site.baseurl }}/images/demo.gif)

It is so cool! It can simulate a real user working with a browser. While many web scraping programs do use a real web browser for data extraction, in most cases the browser they use is WebBrowser Control, which is Internet Explorer. Selenium's WebDriver, however, works with a variety of browsers. It can scrape complicated web pages with dynamic content (like Kickstarter.com). It can even take screenshots of the webpage. It can be cumbersome, but a great web-scraping tool nevertheless.  

Selenium allowed me to access each of Kickstarter’s 15 category pages, where I scraped links to the projects within that category. The tricky part was to make Selenium scroll down the page, so that I could load more than 20 (default) projects. I was able to do it with these few lines of code (I also had to manually activate the browser window to load more than 2 pages worth of projects):

```
   time.sleep(2)
   driver.find_element_by_class_name('load_more').click()
   driver.execute_script("window.scrollTo(0,2500)")
   time.sleep(2)
   driver.execute_script("window.scrollTo(0,5000)")
   time.sleep(12)
   driver.execute_script("window.scrollTo(0,0)")
   time.sleep(2)
```

### Assessing the Catch ###
I was able to scrape 3,169 projects overnight. When I looked at the results, however, :dramatic music: I realized that project launched date wasn’t available from the “surface” scrape - I needed to click on the Updates link within the page, like so:
```
driver.find_element_by_css_selector("a[href*='updates']").click()
time.sleep(2)

try:
    project_dict4['Launched'] = str(driver.find_element_by_class_name('timeline__divider_content').text).split("\n")[0]
except:
    pass
```
Finally, after a couple of days of web-scraping, I have got my data. But it needed much TLC - parsing, recoding, etc., because it came as strings like an example below:
```
'$19,846\npledged of $60,000 goal\n98\nbackers\n23\ndays to go'
```

 Upon splitting, stripping, and changing data types I had a nice list of project features to model on: goal amount, number of backers, days left before expiry date, pledge time window between launch and expiry date, “Projects We Love” badge (something Kickstarter assigns), and country indicator.

### Fitting a Model ### 

A first pass, straight-up version of the model turned out to explain only 25% variation in pledged amount and many of listed features were not significant. Upon another round of explanatory analysis, I have recoded pledge window into an indicator that it was no more than 30 days.

This version of the model explained 75% of variation in project’s pledged amount. This model had just three inputs: number of backers, days left to pledge, and the pledge window less than 30 days indicator. Number of backers were increasing pledged amount (not surprisingly) while the other two inputs have negative impact on it.

Here is a chart of observed pledged amounts versus predicted by model with three inputs:

![Model]({{ site.baseurl }}/images/stations_1.png)

### Trying Something Else ###
The big mass of data points near the y-axis, made me think that my target varies much more than my inputs and one way to correct for it is to build separate models for smaller pledged amounts and larger ones. The distribution of the dependent variable indicated that $1,000 is a good threshold. So I built two more models. The smaller pledged amounts turned out to be dependent on number of backers only, while larger ones were influenced by “Projects we love” badge and location, in addition to days left and pledge window less than 30 days indicator.

Below are the predictions versus observed pledged amounts from the two models:

![Model1]({{ site.baseurl }}/images/le1k_adj.png)

![Model2]({{ site.baseurl }}/images/gt1k_adj.png)


:metal:
Talking about drama and "tormented"! Just kidding. Would be great to add text analysis to this project later. Could be a great insight into what makes people click (or is it "kick"?).
Also, you may appreciate the tongue-in-cheek title of this post. No? Tough crowd, heh.

I can't wait to see what the next project is gonna be. My money is on Vic Mackey.
