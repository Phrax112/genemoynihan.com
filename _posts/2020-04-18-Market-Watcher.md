---
title: Market Watcher in Python
author: Gene
layout: post
---

https://github.com/Phrax112/MarketWatcher

I recently came a cool daily stock market emailer called [ *Bullish* ](https://bullish.email/) on HN.

However, despite the nice formatting I wanted to create something similar which would allow me to tailor the updates to stocks I was interested/invested in. Given I've been learning Python over the last couple of weeks this seemed like a good one day project to hone some skills.

Aside, most sentences of this blog can be read with the prefix "After some googling...". For the sake of the reader and my fingers I will leave it out from this point on. One other thing, all of this was done using Python 3.7.7 on Mac OS Catalina.

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
