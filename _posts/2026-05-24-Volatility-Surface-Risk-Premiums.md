---
layout: post
title: "Volatility Surface Risk Premiums"
date: 2026-05-24
---

## Intro

Hi all, this is my first shot at writing and hopefully something I will continue to do. I plan on adding many posts about volatility, derivatvies and other quant finance related topics and maybe some cs and math topics as well. I am by no means a seasoned proffesional and currently still in my learning journey so if you see something in any of my posts you disagree with or think is blatantly wrong please share with me, also if you just want to reach out and connect my dms on LinkedIn and X are always open. If you are a seasoned proffesional and did somehow end up here please don't hesitate to call me out on any stupid things I may be saying, this is a learning journey and I hope myself and the readers can get something out of it. All of the code and charts in this post can be found on my github, you can look at Volatility-Surface-Risk-Premiums for results related to this post and py_op for the backend calculations. Now onto the good stuff, were going to look into risk premiums specifically related to volatility surfaces.

## Premiums
In investing and trading a risk premium is some type of compensation investors recieve for taking on some type of risk, It is commonly known and persists historically. One of my favorite definitions of a risk premium comes from Author and vol expert Euan Sinclair who describes it as picking up a $20 bill in the middle of the highway, while most people know it is there they may not want to take the risk to go an pick it up, though those who do can be compensated if they don't get run over. 

In general there are four things that need to occur in order for something to be a risk premium. 

1. It needs to persist over time, 

2. It needs to occurr over many asset classes.

3. It needs to be scalable in size

4. There needs to be a economic rationale to support it. 

The most famous risk premium is probably the equity risk premium which is the reward investors get for buying stocks rather than parking your money in a risk free asset or bank account.

There are numerous risk premiums related to the volatility surface, if we can derive a moment related to the risk neutral density and the realized log return distribution there is a risk premium related to it. In this post I want to discuss the risk premium related to the first four moments of the distribution and metrics used to quantify them. 

These are the variance risk premium (vrp), skew risk premium, skewness risk premium and vol of vol risk premium (vvrp). Many different sources will interchange skew and skewness and some consider them seperate things so it is hard to pin down a canonical definition for them. In this post I consider skewness to be related to the shape of a distribution and skew related to the slope of the implied volatility curve or the spot-vol covariance.

## Variance Risk Premium

In the world of options and volatility this is by far the most commonly known risk premium and is measured by the difference in implied volatility and realized volatility. We can measure realized volatility as the sume of the squared log returns and for the implied component I find it is best to use the fixed leg of the corresponding maturities variance swap which is approximated using a list of otm puts and calls.

In the below images we have the 21 day rolling realized volatility and three metrics for our implied part, a rolling iv, the rolling fixed leg of a variance swap and the instantaneous vol calculated from the GVV model (more on this in a future post). These are all calculated from a linear interpolation of the 30 dte closest maturity options.

![Realized Volatility](/assets/images/SPY-Realized-Implied-Volatility-Risk-Premium-Post-2026-05-24.png)

We then find the rolling volatility risk premium using the difference between the fixed leg variance swap strike, and the rolling realized volatility. Note we shift back the realized volatility back to match the date projected by the implied. 

![Volatility Risk Premium](/assets/images/SPY-Variance-Risk-Premium-Post-2026-05-24.png)