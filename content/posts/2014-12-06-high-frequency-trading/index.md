---
title: How do High Frequency Traders Make Their Money?
tags:
- book-notes
- finance
aliases:
- /2014/12/06/high-frequency-trading.html
---
Michael Lewis offers a good story plus a layman introduction to high-frequency trading.

High Frequency Trading is quite a hot topic nowadays. It is a form of
algorithmic trading where you gain an advantage by being faster than
the other market participants. But how do high frequency traders make
their money? The book *Flash Boys* by Michael Lewis, that I’ve
recently read, is a good introduction to the topic. In this post I'll
summarize what I've learned from it.

![Flash Boys](/archive/2014-12-Flash-Boys.jpg)


# HFT Strategies

The important thing is that the HFT firms do not invest, they do a
short term (millisecond) speculations. They risk only small amount of
money (relatively to the money they can win) and they never take
positions over-night. According to the book there are three major ways
in which HFT works.

## 1. Electronic front running

An HFT firm sees an investor’s action on one stock exchange and races
him to the other. For example, the investor wants to buy 100.000
AAPL. Because of the size of the order, it is usually impossible to do
it in one transaction on one exchange. The firm notices the first
small transaction and expects more, larger ones to follow. Therefore,
the firm quickly (quicker than the investor and other firms) buys the
AAPL shares to sell them to the investor at higher price a few
microseconds later.

How can an HFT firm know the investor’s intentions, though? There are
many ways - the simplest and most reliable is to buy access to the
order stream directly from a stock exchange. Firms pay a lot of money
to see the stream of orders just before the other investors. The other
option may be to flood the exchanges with tons of orders to buy and
sell small quantities and through observing their execution predict
the following parts of the transactions. Why does it work? The
broker-dealers are legally obliged to execute the client’s orders at
the best price, even if it means splitting them.

For example, you want to sell 10.000 GOOG. There is a buyer for the
full amount (10.000) on NASDAQ at $500.95 and another buyer for 200 at
$500.96 on a different exchange. Your broker will first sell 200 for
$500.96/share (she has to do it), and then order selling the remaining
9800 on NASDAQ. However, by the time her order reaches NASDAQ, the
10.000 has got his shares from the HTF firms, and that is them to buy
yours at a lower price.

![Electronic front running](/archive/2014-12-front-running.jpg)


## 2. Rebate arbitrage

In the old days there were one or two stock exchanges per country with
a flat commission rate paid to them for executing orders. Nowadays,
there are many of them (over fifteen public exchanges in the US plus
even more private exchanges, aka dark pools). Instead of a flat rate
they offer sophisticated schemes of commissions and premiums paid by
and given to the trading parties. The HFT firms buy and sell in a way
which provides no additional liquidity, but scalps the premiums from
the exchanges. For example, an exchange A pays a small premium for
buying and an exchange B for selling. The HFT can race both sides of
the deal to the exchanges and take the premiums.

## 3. Slow market arbitrage

According to the book, this is where the most of the money is
made. The idea is simple, a company is traded at $100 at all of the
exchanges; then a big order arrives at one of them which pushes the
price a bit, say to $100.01. The fastest player can buy at $100 and
sell at $100.01 taking virtually no risk.

![Slow market arbitrage](/archive/2014-12-slow-market.jpg)

## Money at stake

High frequency trading firms are new kind of beasts in the financial
markets. They are not on the *sell side*, like investment banks
creating new vehicles to invest in; they are not on the *buy side*
either, like mutual funds picking the best assets. They are more like
a tax the investors have to pay. Due to various features of the
markets they are almost guaranteed to outrun all the regular investors
(including even the biggest funds). They boast many years without
losing money even a single day.  They take small risks and earn huge
profits at the expense of the other investors.

In the book the tax is estimated at 0.064% (a bit more than six
hundredth of percent) of a trading volume. In 2009 it gave $160
million per day on the US stock market.


## The challenge

As an HFT you need to be first. The trading algorithms themselves are
not exceptionally sophisticated in this business. What does matter is
the raw speed. Each microsecond counts. It takes 300-400 microseconds
to blink an eye ([src][1]), while high frequency traders are interested in
gaining an advantage hundred times smaller. To achieve such a speed
you need to be crazy about your hardware, connection lags to the stock
exchange point of entry, routers, cables, everything. Of course your
software also needs to be tuned for speed.

Even though, the book portraits HFT as a modern equivalent of a
highwayman and their trade as a morally doubtful one, from the
technological point of view it is a very interesting problem and a
huge challenge. I envy a bit the engineers working there.

## Book

This book is heavily criticized by some people in mass and social
media. Of course not everybody will like it. Nevertheless, a lot of
the criticism comes from the people having a stake in HFT market, who
therefore have good incentive to keep it obscure ([src][2]).

In my opinion, the book is great. It offers an interesting story with
both positive characters and villains. You don’t need an MIT degree to
follow it and yet it presents the big picture of high frequency
trading in a clear and manner. It's worth your time and attention if
you're interested even a little bit in financial markets, the
technology behind them and the associated ethical challenges. It also
shows a new kind of industry making great money and the attempts to
tame them. I can highly recommend it.




[1]: http://www.madsci.org/posts/archives/nov98/911697403.Me.r.html
[2]: http://blog.themistrading.com/1215095-the-flash-boys-mystery-solved/

