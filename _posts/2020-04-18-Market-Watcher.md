---
title: Market Watcher in Python
author: Gene
layout: post
---

[ *Github link for discussed code* ](https://github.com/Phrax112/MarketWatcher)

I recently came across a cool daily stock market emailer called [ *Bullish* ](https://bullish.email/) on HN. However, despite the nice formatting I wanted to create something similar which would allow me to tailor the updates to stocks I was interested/invested in. Given I've been learning Python over the last couple of weeks this seemed like a good one day project to learn some skills.

<h3>Aside</h3>
Most sentences of this blog can be read with the prefix "After some googling..." . For the sake of the reader and my fingers I will leave it out from this point on. One other thing, all of this was done using Python 3.7.7 on Mac OS Catalina.

It appears the Yahoo Finance API has been deprecated but is still available for use if you know how (or your library of choice does). Given I simply wanted historic closing prices for a small selection of dates I decided to stick with this particular provider as I have found it the best in the past. The [ *yfinance* ](https://pypi.org/project/yfinance/) library is a nice choice for this kind of action as it allows you to pass multiple symbols at once and comes with a whole host of other functionality if you did find yourself wanting to include market calendars, dividends etc... down the line. All market data is downloaded as a handy dataframe for further manipulation.

<h3>yfinance Example</h3>
<pre><code>

>>> import yfinance as yf
>>> res = yf.download('HSBA.L', start='2020-04-12', end='2020-04-17')
[*********************100%***********************]  1 of 1 completed
>>> res
                  Open        High  ...   Adj Close    Volume
Date                                ...                      
2020-04-14  429.049988  432.001007  ...  427.450012  36035662
2020-04-15  421.950012  422.200012  ...  408.100006  55587634
2020-04-16  410.700012  412.700012  ...  405.600006  37932065

[3 rows x 6 columns]
>>> res["Adj Close"]
Date
2020-04-14    427.450012
2020-04-15    408.100006
2020-04-16    405.600006
Name: Adj Close, dtype: float64

</code></pre>

Now that I could get the data for a selection of tickers and dates I just have to iterate over these two loops to get all of the information that I need. One thing to note is that given my personal selection of stocks is quite small (~20) I make each date call separately although there is grouping on the tickers. I also brute force ignoring market closure days by always grabbing a small range (this is highlighted below). This can obviously be made more efficient but for my purposes and this example works just fine

<h3>Retrieving data for specified tickers and dates</h3>
<pre><code>
>>> import pandas as pd
>>> import datetime as dt
>>> closing_prices = pd.DataFrame()
>>> TICKERS = ['^FTSE', 'VOD.L', 'HSBA.L']
>>> INTERVALS = [0, 1, 7, 30, 90, 180, 365]
>>> last_working_date = dt.date.today() - dt.timedelta(days = 1)
>>> str(dt.date.today()) # today is a Saturday
'2020-04-18'
>>> last_working_date = dt.date.today() - dt.timedelta(days = 1)
# yesterday is a friday so we are fine, in the script we do account for weekends
>>> str(last_working_date)
'2020-04-17'
>>> for i in INTERVALS:
...     date = last_working_date - dt.timedelta(days = i) # get the date at the start of the interval we want
...     print("Retrieving data for date:", date)
        # download the data for that date, + a 4 day period before (brute force market closure problems)
...     res = yf.download(TICKERS, start=date - dt.timedelta(days=4), end=date + dt.timedelta(days=1))[-1:]
...     closing_prices = results.append(res["Adj Close"])
...
Retrieving data for date: 2020-04-17
[*********************100%***********************]  3 of 3 completed
Retrieving data for date: 2020-04-16
[*********************100%***********************]  3 of 3 completed
Retrieving data for date: 2020-04-10
[*********************100%***********************]  3 of 3 completed
Retrieving data for date: 2020-03-18
[*********************100%***********************]  3 of 3 completed
Retrieving data for date: 2020-01-18
[*********************100%***********************]  3 of 3 completed
Retrieving data for date: 2019-10-20
[*********************100%***********************]  3 of 3 completed
Retrieving data for date: 2019-04-18
[*********************100%***********************]  3 of 3 completed
>>> closing_prices
                HSBA.L       VOD.L        ^FTSE
Date                                           
2020-04-17  414.350006  108.879997  5787.000000
2020-04-16  405.600006  107.160004  5628.399902
2020-04-09  428.850006  113.019997  5842.700195
2020-03-18  491.000000  107.379997  5080.600098
2020-01-17  571.524292  154.380005  7674.600098
2019-10-18  578.836731  155.978882  7150.600098
2019-04-18  617.184387  134.230789  7459.899902

</code></pre>

Now we have all of the required data we just need to calculate the differences between each row and the first row i.e. the change in price over all of our required intervals. As we want to keep things pretty tidy I also found it useful to set the pandas precision to just two decimal places

<pre><code>

>>> pd.options.display.float_format = '{:.2f}'.format
>>> diff_results = pd.DataFrame()
>>> diff_results["INTERVALS"] = INTERVALS
>>> diff_results = diff_results.set_index('INTERVALS')
>>> for nm in closing_prices:
...     diffs = []
...     for prc in closing_prices[nm]:
...             diffs.append( 100 * (closing_prices[nm][0] - prc)/prc )
...     diff_results[nm] = diffs
...
>>> diff_results
           HSBA.L  VOD.L  ^FTSE
INTERVALS                      
0            0.00   0.00   0.00
1            2.16   1.61   2.82
7           -3.38  -3.66  -0.95
30         -15.61   1.40  13.90
90         -27.50 -29.47 -24.60
180        -28.42 -30.20 -19.07
365        -32.86 -18.89 -22.43
>>>

</code></pre>

Now we just need to send out our tables! I converted them to html using the standard 'to_html' function for a dataframe and then used the smtplib and email libraries to create the message and send it via the gmail SMTP server. While the connection is secured using the SSL library I would highly recommend creating a separate email account for the use of sending the emails. I simply created a new gmail account for example as you will need to store the password somewhere when the script is going to run.

For the information on what all of these variables point to just check the Github link above for the full code, this is just an example.

<h3>Sending an email</h3>
<pre><code>
>>> message = MIMEMultipart("alternative")
>>> message["Subject"] = MSG_TITLE
>>> message["From"] = SENDER_EMAIL
>>> message["To"] = REC_EMAIL
>>> msg = closing_prices[0:1].to_html(formatters={'Name': lambda x:   '<b>' + x + '</b>'})
>>> msg = MIMEText(msg, "html")
>>> message.attach(msg)
>>> context = ssl.create_default_context()
>>> with smtplib.SMTP_SSL("smtp.gmail.com", TLS_PORT, context=context) as server:
...     server.login(SENDER_EMAIL, EMAIL_PASS)
...     server.sendmail(SENDER_EMAIL, REC_EMAIL, message.as_string())

</pre></code>

When all is said and done this is an example of the email I get in my inbox every morning once I've set up this script to run on a cron job on my raspberry pi. The formatting is pretty ugly as it's currently just raw HTML but some prettifying is pretty straight forward with a style sheet. Plus, the uglier it is the less time I spent looking at it and thinking how bad my investments are!

![Email Message](assets/images/marketwatcher_example.png)
