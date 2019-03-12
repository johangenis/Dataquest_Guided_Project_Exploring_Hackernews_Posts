
# Exploring Hacker News Posts
![hacker_news.jpg](attachment:hacker_news.jpg)

[Hacker news](https://news.ycombinator.com/) is a site started by the startup incubator [Y Combinator](https://www.ycombinator.com/), where user-submitted stories (known as 'posts') are voted and commented upon, similar to reddit. Hacker News is extremely popular in technology and startup circles, and posts that make it to the top of Hacker News' listings can get hundreds of thousands of visitors as a result.

You can find the data set [here](https://www.kaggle.com/hacker-news/hacker-news-posts), but note that it has been reduced from almost 300,000 rows to approximately 20,000 rows by removing all submissions that did not receive any comments, and then randomly sampling from the remaining submissions.

Below are descriptions of the columns:

* id - The unique identifier from Hacker News for the post
* title - The title of the post
* url - The URL that the posts links to, if it the post has a URL
* num_points - The number of points the post acquired, calculated as the total number of upvotes minus the total number of downvotes
* num_comments - The number of comments that were made on the post
* author - The username of the person who submitted the post
* created_at - The date and time at which the post was submitted

We'll compare **Ask HN** and **Show HN** posts to determine the following:

* Do **Ask HN** or **Show HN** receive more comments on average?
* Do posts created at a certain time receive more comments on average?

## Introduction

We'll start by reading in the data and removing the headers:


```python
# Read the data in
import csv

f = open('hacker-news.csv')
hn = list(csv.reader(f))
hn[:5]
```




    [['id', 'title', 'url', 'num_points', 'num_comments', 'author', 'created_at'],
     ['12579008',
      'You have two days to comment if you want stem cells to be classified as your own',
      'http://www.regulations.gov/document?D=FDA-2015-D-3719-0018',
      '1',
      '0',
      'altstar',
      '9/26/2016 3:26'],
     ['12579005',
      'SQLAR  the SQLite Archiver',
      'https://www.sqlite.org/sqlar/doc/trunk/README.md',
      '1',
      '0',
      'blacksqr',
      '9/26/2016 3:24'],
     ['12578997',
      'What if we just printed a flatscreen television on the side of our boxes?',
      'https://medium.com/vanmoof/our-secrets-out-f21c1f03fdc8#.ietxmez43',
      '1',
      '0',
      'pavel_lishin',
      '9/26/2016 3:19'],
     ['12578989',
      'algorithmic music',
      'http://cacm.acm.org/magazines/2011/7/109891-algorithmic-composition/fulltext',
      '1',
      '0',
      'poindontcare',
      '9/26/2016 3:16']]



## Removing Headers from the Hacker News List of Lists


```python
# Remove the headers
```


```python
headers = hn[0]
hn = hn[1:]
print(headers)
print(hn[:5])
```

    ['id', 'title', 'url', 'num_points', 'num_comments', 'author', 'created_at']
    [['12579008', 'You have two days to comment if you want stem cells to be classified as your own', 'http://www.regulations.gov/document?D=FDA-2015-D-3719-0018', '1', '0', 'altstar', '9/26/2016 3:26'], ['12579005', 'SQLAR  the SQLite Archiver', 'https://www.sqlite.org/sqlar/doc/trunk/README.md', '1', '0', 'blacksqr', '9/26/2016 3:24'], ['12578997', 'What if we just printed a flatscreen television on the side of our boxes?', 'https://medium.com/vanmoof/our-secrets-out-f21c1f03fdc8#.ietxmez43', '1', '0', 'pavel_lishin', '9/26/2016 3:19'], ['12578989', 'algorithmic music', 'http://cacm.acm.org/magazines/2011/7/109891-algorithmic-composition/fulltext', '1', '0', 'poindontcare', '9/26/2016 3:16'], ['12578979', 'How the Data Vault Enables the Next-Gen Data Warehouse and Data Lake', 'https://www.talend.com/blog/2016/05/12/talend-and-Â\x93the-data-vaultÂ\x94', '1', '0', 'markgainor1', '9/26/2016 3:14']]


We can see above that the data set contains the title of the posts, the number of comments for each post, and the date the post was created. Let's start by exploring the number of comments for each type of post.

## Extracting Ask HN and Show HN Posts
First, we'll identify posts that begin with either Ask HN or Show HN and separate the data for those two types of posts into different lists. Separating the data makes it easier to analyze in the following steps.


```python
# Identify posts that begin with either `Ask HN` or `Show HN` and separate the data into different lists.
ask_posts = []
show_posts =[]
other_posts = []

for post in hn:
    title = post[1]
    if title.lower().startswith("ask hn"):
        ask_posts.append(post)
    elif title.lower().startswith("show hn"):
        show_posts.append(post)
    else:
        other_posts.append(post)
        
print(len(ask_posts))
print(len(show_posts))
print(len(other_posts))
```

    9139
    10158
    273822


# Calculating the Average Number of Comments for Ask HN and Show HN Posts
Now that we separated ask posts and show posts into different lists, we'll calculate the average number of comments each type of post receives.


```python
# Calculate the average number of comments `Ask HN` posts receive.
total_ask_comments = 0

for post in ask_posts:
    total_ask_comments += int(post[4])
    
avg_ask_comments = total_ask_comments / len(ask_posts)
print(avg_ask_comments)

total_show_comments = 0

for post in show_posts:
    total_show_comments += int(post[4])
    
ave_show_comments = total_show_comments / len(show_posts)
print(ave_show_comments)
```

    10.393478498741656
    4.886099625910612


On average, ask posts in our sample receive approximately 10 comments, whereas show posts receive approximately 4. Since ask posts are more likely to receive comments, we'll focus our remaining analysis just on these posts.

# Finding the Amount of Ask Posts and Comments by Hour Created
Next, we'll determine if we can maximize the amount of comments an ask post receives by creating it at a certain time. First, we'll find the amount of ask posts created during each hour of day, along with the number of comments those posts received. Then, we'll calculate the average amount of comments ask posts created at each hour of the day receive.


```python
# Calculate the amount of ask posts created during each hour of day and the number of comments received.
import datetime as dt

result_list = []

for post in ask_posts:
    result_list.append(
        [post[6], int(post[4])]
    )

comments_by_hour = {}
counts_by_hour = {}
date_format = "%m/%d/%Y %H:%M"

for each_row in result_list:
    date = each_row[0]
    comment = each_row[1]
    time = dt.datetime.strptime(date, date_format).strftime("%H")
    if time in counts_by_hour:
        comments_by_hour[time] += comment
        counts_by_hour[time] += 1
    else:
        comments_by_hour[time] = comment
        counts_by_hour[time] = 1

comments_by_hour
```




    {'02': 2996,
     '01': 2089,
     '22': 3372,
     '21': 4500,
     '19': 3954,
     '17': 5547,
     '15': 18525,
     '14': 4972,
     '13': 7245,
     '11': 2797,
     '10': 3013,
     '09': 1477,
     '07': 1585,
     '03': 2154,
     '23': 2297,
     '20': 4462,
     '16': 4466,
     '08': 2362,
     '00': 2277,
     '18': 4877,
     '12': 4234,
     '04': 2360,
     '06': 1587,
     '05': 1838}



# Calculating the Average Number of Comments for Ask HN Posts by Hour


```python
# Calculate the average amount of comments `Ask HN` posts created at each hour of the day receive.
avg_by_hour = []

for hr in comments_by_hour:
    avg_by_hour.append([hr, comments_by_hour[hr] / counts_by_hour[hr]])

avg_by_hour
```




    [['02', 11.137546468401487],
     ['01', 7.407801418439717],
     ['22', 8.804177545691905],
     ['21', 8.687258687258687],
     ['19', 7.163043478260869],
     ['17', 9.449744463373083],
     ['15', 28.676470588235293],
     ['14', 9.692007797270955],
     ['13', 16.31756756756757],
     ['11', 8.96474358974359],
     ['10', 10.684397163120567],
     ['09', 6.653153153153153],
     ['07', 7.013274336283186],
     ['03', 7.948339483394834],
     ['23', 6.696793002915452],
     ['20', 8.749019607843136],
     ['16', 7.713298791018998],
     ['08', 9.190661478599221],
     ['00', 7.5647840531561465],
     ['18', 7.94299674267101],
     ['12', 12.380116959064328],
     ['04', 9.7119341563786],
     ['06', 6.782051282051282],
     ['05', 8.794258373205741]]




```python
# Calculate the average amount of comments `Ask HN` posts created at each hour of the day receive.
avg_by_hour = []

for hr in comments_by_hour:
    avg_by_hour.append([hr, comments_by_hour[hr] / counts_by_hour[hr]])

avg_by_hour
```




    [['02', 11.137546468401487],
     ['01', 7.407801418439717],
     ['22', 8.804177545691905],
     ['21', 8.687258687258687],
     ['19', 7.163043478260869],
     ['17', 9.449744463373083],
     ['15', 28.676470588235293],
     ['14', 9.692007797270955],
     ['13', 16.31756756756757],
     ['11', 8.96474358974359],
     ['10', 10.684397163120567],
     ['09', 6.653153153153153],
     ['07', 7.013274336283186],
     ['03', 7.948339483394834],
     ['23', 6.696793002915452],
     ['20', 8.749019607843136],
     ['16', 7.713298791018998],
     ['08', 9.190661478599221],
     ['00', 7.5647840531561465],
     ['18', 7.94299674267101],
     ['12', 12.380116959064328],
     ['04', 9.7119341563786],
     ['06', 6.782051282051282],
     ['05', 8.794258373205741]]




```python
swap_avg_by_hour = []

for row in avg_by_hour:
    swap_avg_by_hour.append([row[1], row[0]])

print(swap_avg_by_hour)

sorted_swap = sorted(swap_avg_by_hour, reverse=True)

sorted_swap
```

    [[11.137546468401487, '02'], [7.407801418439717, '01'], [8.804177545691905, '22'], [8.687258687258687, '21'], [7.163043478260869, '19'], [9.449744463373083, '17'], [28.676470588235293, '15'], [9.692007797270955, '14'], [16.31756756756757, '13'], [8.96474358974359, '11'], [10.684397163120567, '10'], [6.653153153153153, '09'], [7.013274336283186, '07'], [7.948339483394834, '03'], [6.696793002915452, '23'], [8.749019607843136, '20'], [7.713298791018998, '16'], [9.190661478599221, '08'], [7.5647840531561465, '00'], [7.94299674267101, '18'], [12.380116959064328, '12'], [9.7119341563786, '04'], [6.782051282051282, '06'], [8.794258373205741, '05']]





    [[28.676470588235293, '15'],
     [16.31756756756757, '13'],
     [12.380116959064328, '12'],
     [11.137546468401487, '02'],
     [10.684397163120567, '10'],
     [9.7119341563786, '04'],
     [9.692007797270955, '14'],
     [9.449744463373083, '17'],
     [9.190661478599221, '08'],
     [8.96474358974359, '11'],
     [8.804177545691905, '22'],
     [8.794258373205741, '05'],
     [8.749019607843136, '20'],
     [8.687258687258687, '21'],
     [7.948339483394834, '03'],
     [7.94299674267101, '18'],
     [7.713298791018998, '16'],
     [7.5647840531561465, '00'],
     [7.407801418439717, '01'],
     [7.163043478260869, '19'],
     [7.013274336283186, '07'],
     [6.782051282051282, '06'],
     [6.696793002915452, '23'],
     [6.653153153153153, '09']]




```python
# Sort the values and print the the 5 hours with the highest average comments.

print("Top 5 Hours for 'Ask HN' Comments")
for avg, hr in sorted_swap[:5]:
    print(
        "{}: {:.2f} average comments per post".format(
            dt.datetime.strptime(hr, "%H").strftime("%H:%M"),avg
        )
    )
```

    Top 5 Hours for 'Ask HN' Comments
    15:00: 28.68 average comments per post
    13:00: 16.32 average comments per post
    12:00: 12.38 average comments per post
    02:00: 11.14 average comments per post
    10:00: 10.68 average comments per post


The hour that receives the most comments per post on average is 15:00, with an average of 38.59 comments per post. There's about a 60% increase in the number of comments between the hours with the highest and second highest average number of comments.

Conclusion
In this project, we analyzed ask posts and show posts to determine which type of post and time receive the most comments on average. Based on our analysis, to maximize the amount of comments a post receives, we'd recommend the post be categorized as ask post and created between 15:00 and 16:00.

However, it should be noted that the data set we analyzed excluded posts without any comments. Given that, it's more accurate to say that of the posts that received comments, ask posts received more comments on average and ask posts created between 15:00 and 16:00 received the most comments on average.


```python

```
