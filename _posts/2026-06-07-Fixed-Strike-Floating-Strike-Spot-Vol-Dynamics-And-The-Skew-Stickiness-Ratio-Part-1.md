---
layout: post
title: "Fixed Strike Volatility, Floating Strike Volatility, Spot Vol Dynamics And The Skew Stickiness Ratio Part 1"
date: 2026-06-07
---

## Intro
Today I want to talk about some topics in the volatility space that confused me for a long time but are extremely important to understanding volatility modeling and volatility trading nomenclature. The terms fixed strike, floating strike, sticky strike and sticky delta (or alternative version) come up a lot all over the place and are sometimes thrown around loosley. On top of that if we throw fixed strike skew and floating strike skew which we can also replace skew with spot vol beta cov, corr, etc... You can see how these terms can get confusing. I still top this day will see a pretty chart or formula saying fixed strike skew term structure and have to stop and stare at it for a second. I've seen many well established successful people in the vol space mix them up and confuse one with the other and that is part of the reason why they can be tricky to get a grasp of, so what I want to do in this post is talk about how I like to think of them and give some examples of there usage in the world of volatility.

## Definitions
I am going to start off by simply defining these terms and maybe giving you a few pictures to hammer home the meanings. A great way to think about these which was elequantly defined by Benn Eifert from QVR is "Sticky strike and sticky delta refer to spot vol dynamics and fixed stirke and floating strike vol are specific parameters of the volatility surface". Not sure if this is an exact quote but close enough. That definition is something you should keep in your head especially as we move forward thoughout this post (and in future posts) and I will often go back to it.

### Fixed Strike - Floating Strike Volatility
What does it mean to parameterize a volatility surface, well in mathematical terms we can represent the volatility surface as some function $\sigma(.)$ that has specific parameters that when we plug them in we get an implied volatility. On a fixed strike basis the parameters would be strike K and maturity T $\sigma(K, T)$. In general I like to think about the parameterization of the volatility suraface as how we define the x-axis, for fixed strike we use our strikes and for floating strike it would be something like delta $\sigma(\Delta, T)$ or moneyness $\sigma(k, T)$ ($k = K / F$).

What really helped me understand these topics is reading through Emanuel Dermans volatility class lectures. In the lectures he explores fixed strike and floating strike vol from a time series perspective and I want to show that here. Below I show a time series of multiple fixed strike volatilities, that means at the starting date we choose an option with a specific strike and plot how that same strike option changes everyday, in our example we start at 2026-01-01 and set the strike to be 660 on the lower end and 700 on the higher end when spot is at 680. Every day going forward we plot the same IV for those strike options keeping them fixed.

![Fixed Strike Vol](/assets/images/SPY_Fixed_Strike_Vol_2026-06-30.png)

Also it should be noted just like fixed and floating strike we can have a constant maturity (dte) or expiration date and we show that above. A constant expiration is the same expiration date and therefore same option contract throughout the whole time series whereas for constant maturity it is the same days to expiration for the whole series. This is extremely important with respects to backtesting and strategy rebalancing so it is important to be cognizant of that.

Next we show similar graphs for a floating strike volatility time series. Remember because of the specific parameterization of the volatiltiy surface (in this case moneyness or delta) every iv on the time series refers to a different strike

![Floating Strike Vol](/assets/images/SPY_Floating_Strike_Vol-2026-06-30.png)

Usually when someone plots a rolling chart of some implied volatility metric it is usually a constant maturity floating strike volatility. For example if you ever see someone post a chart of the ATM IV it is 99% of the time a constant maturity floating stirke ATM IV but nobody ever defined it like that. Another common example is the VIX, this is a constant maturity floating strike volatility metric. the disticntion between fixed and floating strike IV is hardly made but it is important as the below example illustrates.

If you are trying to trade a pure long volatility position by, lets say, buying a straddle and delta hedging it, you usually start buy buying the ATM put and call. The issue is positions and trades are not floating strike, they have to be discretely rehedged to become semi floating strike. If you buy a delta hedged straddle today and dont look at it for a week more likely then not the original spot price will have moved away from ATM and it is no longer ATM anymore. On fixed strike basis how vol chnages is hugely important to our positions P&L, did fixed strike follow the skew curve, did the fixed strike vol meaningfully outperform the skew curve. These are all things options traders look at and need to know and it brings us to our next topic.

### Spot Vol Dynamics
Definition: Mention 3 regimes from collin bennet and SSR briefly

One hot code:

Out skew Curves

SSR:

### Risk Reversal 3 Scenarios

In Progress -- Stay Tuned