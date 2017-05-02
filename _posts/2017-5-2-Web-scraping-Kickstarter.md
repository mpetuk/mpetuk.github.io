---
“Luther is a brilliant but emotionally impulsive detective who is tormented by the dark side of humanity while hunting down murderers.”
---

I love it that Metis names our projects after characters of different drama series and that degree of drama raises, appropriately, each week. As such, we went from “Law and Order SVU” (week 1) to “Luther” (week 2)  to “The Wire” (week 3). 

![_config.yml]({{ site.baseurl }}/images/Luther.jpg)  #picture of Luther here

And this story is about Project Luther.  The task was to come up with a question that can be answered by means of linear regression algorithm. Sounds easy enough, right? But wait - there is more! The data for the project was to be web-scaped.

I decided that Kickstarter.com is a good candidate with goal and pledged amounts and other project characteristics available and that it would be interested to try to predict project success by predicting its pledged amount. 

Quick refresher: Kickstarter is a platform for creative projects to get crowd-funded. If project’s goal met within given time period, pledges become real funds. 

Kickstarter’s web site is built using XML data, which makes it difficult to scrape, but in the end, Selenium did the work. 

Selenium is a suite of tools to automate web browsers across many platforms. It allows, among other things, access web sites and interact with their content from Python, for example. 

It allowed me to access each of 15 categories pages and scraped the links to the projects within. The tricky part was to make  Selenium scroll the page, so that I could load more than 20 (default) projects.  I was able to do it with this few lines of code:

    time.sleep(2)
    driver.find_element_by_class_name('load_more').click()
    driver.execute_script("window.scrollTo(0,2500)")
    time.sleep(2)
    driver.execute_script("window.scrollTo(0,5000)")
    time.sleep(12)
    driver.execute_script("window.scrollTo(0,0)")
    time.sleep(2)

When I looked at the results, I realized that project launched date wasn’t available from the “surface” scrape - I needed to click on the Updates link within the page, like so:

    driver.find_element_by_css_selector("a[href*='updates']").click()
    time.sleep(2)
    
    try:
        project_dict4['Launched'] = str(driver.find_element_by_class_name('timeline__divider_content').text).split("\n")[0]
    except:
        pass

Finally, after a couple days of web-scraping, I had my data. But it needed much TLC - parsing, recoding, etc., upon which I had a nice list of project features to model on: goal amount, number of backers, days left before expiry date, pledge time window between launch and expiry date, “Projects We Love” badge (something Kickstarter assigns), and country indicator.

First, straight-up version of the model, turned out explaining only 25% variation in pledged amount and many of listed features were not significant. Upon a round of explanatory analysis, I have recoded pledge window into an indicator that it was no more than 30 days.  I have also windsorized my target variables to get rid off extremely high values.

This version of the model was explaining 75% of variation in project’s pledged amount. This model had just three inputs: number of backers, days left to pledge, and the pledge window less than 30 days indicator. Number of backers were increasing pledged amount (not surprisingly) while the other two inputs have negative impact on it. 

The big mass of data points near the y-axis, made me think that  my target vary much more than my inputs and one way to correct for it is to build separate models for smaller pledged amounts and larger ones. The distribution of the dependent variable indicated that $1,000 is a good threshold. So I built two more models. The smaller pledged amounts turned out to be dependent on number of backers only, while larger ones were influenced by “Projects we love” badge and location, in addition to days left and pledge window less than 30 days indicator.

Phew! Next steps would be trying to scrape more data and conduct text analytics on project description.

After all this drama, I can't wait to see if the next project is gonna be called after Vic Mackey.


 
