---
layout: post
title: "Volatility Surface Risk Premiums"
date: 2026-05-24
---

## Intro

Hi all, this is my first shot at writing and hopefully something I will continue to do. I plan on adding many posts about volatility, derivatvies and other quant finance related topics and maybe some cs and math topics as well. I am by no means a seasoned proffesional and currently still in my learning journey so if you see something in any of my posts you disagree with or think is blatantly wrong please share with me, also if you just want to reach out and connect my dms on LinkedIn and X are always open. If you are a seasoned proffesional and did somehow end up here please don't hesitate to call me out on any stupid things, this is a learning journey and I hope myself and the readers can get something out of it. All of the code and charts in this post can be found on my github, you can look at Volatility-Surface-Risk-Premiums for results related to this post and py_op for the backend calculations. Now onto the good stuff, were going to look into risk premiums specifically related to volatility surfaces.

## Premiums
In investing and trading a risk premium is some type of compensation investors recieve for taking on some type of risk, It is commonly known and persists historically. One of my favorite definitions of a risk premium comes from Author and vol expert Euan Sinclair who describes it as picking up a $20 bill in the middle of the highway, while most people know it is there they may not want to take the risk to go an pick it up, though those who do can be compensated if they don't get run over. 

In general there are four things that need to occur in order for something to be a risk premium. 

1. It needs to persist over time, 

2. It needs to occur over many asset classes.

3. It needs to be scalable in size

4. There needs to be a economic rationale to support it. 

The most famous risk premium is probably the equity risk premium which is the reward investors get for buying stocks rather than parking their money in a risk free asset or bank account.

There are numerous risk premiums related to the volatility surface, if we can estimate a moment related to the risk neutral density and physical density there is a risk premium related to it. In this post I want to discuss the risk premium related to the first four moments and metrics used to quantify them. 

These are the variance risk premium (vrp), skew risk premium, skewness risk premium and vol of vol risk premium (vvrp). Many different sources will interchange skew and skewness and some consider them seperate things so it is hard to pin down a canonical definition for them. In this post I consider skewness to be related to the shape of a distribution and skew related to the slope of the implied volatility curve or the spot-vol covariance.

## Variance Risk Premium

In the world of options and volatility this is by far the most commonly known risk premium and is measured by the difference in implied volatility and realized volatility. We can measure realized volatility as the sum of the squared log returns and for the implied component I find it is best to use the fixed leg of the corresponding maturities variance swap which is approximated using a list of otm puts and calls.

In the below images we have the 21 day rolling realized volatility and three metrics for our implied part, a rolling iv, the rolling fixed leg of a variance swap and the instantaneous vol calculated from the GVV model (more on this in a future post). These are all calculated from a linear interpolation of the 30 dte closest maturity options.

![Realized Volatility](/assets/images/SPY-Realized-Implied-Volatility-Risk-Premium-Post-2026-05-24.png)

We then find the rolling volatility risk premium using the difference between the fixed leg variance swap strike, and the rolling realized volatility. Note we shift back the realized volatility back to match the date projected by the implied. 

![Volatility Risk Premium](/assets/images/SPY-Variance-Risk-Premium-Post-2026-05-24.png)

I wont talk any more about the vrp because it is widely talked about and researched and I doubt there is much more value to be added. It is important to note that investors/traders usually harvest the variance risk premium using delta hedged straddles.

## Skew/Skewness Risk Premium

If the variance risk premium measures the premia related to dispersion then the skewness/skew risk premium measures the premia related to asymetry. Unlike the vrp there is no agreed upon metric used to quantify this and cannot be measured directly because the conditional physical density is not observable. In this section I will present, in my opinion, a few decent metrics we can use to estimate the skew/skewness risk premium.

Just like the vrp we define the premia as the difference between expected skewness/skew under the risk-neutral measure (implied) and expected skewness/skew under the physical measure (realized). I quantify this in three ways, following the skew-swap framework of Kozhan, Neuberger, and Schneider (2012), the model-free VRP/SRP approach in Ito (2025), and taking the difference between implied - realized spot-vol covariance. Under these frameworks the risk premium can be interpreted as the expected payoff of a skew swap. Most traders capture the premia using option structures like risk reversals or some weighting of short OTM puts and long OTM calls.

We will use the following metrics as our estimates:

###### Implied Skew: 
- Implied spot-vol covaraince: found from an ATM slope of a implied skew curve. I approximate this using calibrated implied parameters from SABR, GVV or some other stochastic volatility model. You can approximate this by multiplying parameters representing spot-vol cov, instantaneous vol and vol of vol.
- Normalized 90% - 110% moneyness IV skew slope
 
  $$
  \frac{\text{90 put IV} - \text{110 call IV}}
  {\left(\text{90 put strike} - \text{110 call strike}\right) / F}
  $$

###### Realized Skew: 
- The historical covariance between instantaneous vol and log returns: 

  $$
  \sum_{i=1}^{N} \ln\left(\frac{S_i}{S_{i-1}}\right)\Delta \hat{\sigma}_i
  $$

- Floating leg of a dynamically rebalanced skew swap, scaled by the initial variance-swap fixed leg

  $$
  \operatorname{rskew}_{t,T} 
  =
  \frac{rs_{t,T}}{\left(v^{L}_{t,T}\right)^{3/2}}
  $$

  $$
  rs_{t,T}
  =
  \sum_{i=t}^{T}
  \left[
  \delta v^{E}_{i,T}
  \left(e^{r_{i,i+1}} - 1\right)
  +
  6
  \left(
  2 - 2e^{r_{i,i+1}}
  + r_{i,i+1}
  + r_{i,i+1}e^{r_{i,i+1}}
  \right)
  \right]
  $$

  - Following Kozhan, Neuberger, and Schneider (2012)

###### Implied Skewness:
- Fixed leg of a skew swap: There are many different ways to estimate this and many papers have been written that go into this topic on length but here I will present a few below.

###### Realized Skewness:
- Third moment of the historical return distribution: 

  $$
  \frac{\sqrt{N}\sum_{i=1}^{N} r_i^3}
  {\left(\sum_{i=1}^{N} r_i^2\right)^{3/2}}
  $$

  - Following Ito (2025)


And we provide three different quantities used to estimate the srp:

1. Implied spot-vol covariance - realized spot-vol covariance

2. Skew swap fixed leg - floating leg

3. Skew swap fixed leg - realized third moment of the historical return distribution

### Fixed Leg of a Skew Swap

I think it is worth pausing briefly to clarify the fixed leg of a skew swap. Above, I wrote that implied skewness is found using the fixed leg of a skew swap, but that is not completely accurate. Some sources use terms like implied skew, implied skewness, and skew swap fixed leg somewhat interchangeably, but the exact interpretation depends on the calculation methodology. Generally the fixed leg of a skew swap is approximated using a strip of OTM puts and calls similar to the fixed leg of a variance swap except with a different weighting scheme. Below I present three different formulas for this approximation

- Model-free implied skewness following Ito (2025), who defines skewness using model-free risk-neutral moments estimated from option prices: 

  $$
  P_1 = \mu = E[R]
  = e^{rT}
  \left(
  -\sum_i \frac{1}{K_i^2} Q_{K_i}\Delta K_i
  \right)
  + \varepsilon_1
  $$

  $$
  P_2 = E[R^2]
  = e^{rT}
  \left(
  \sum_i \frac{2}{K_i^2}
  \left(
  1 - \ln\frac{K_i}{F_0}
  \right)
  Q_{K_i}\Delta K_i
  \right)
  + \varepsilon_2
  $$

  $$
  P_3 = E[R^3]
  = e^{rT}
  \left(
  \sum_i \frac{3}{K_i^2}
  \left\{
  2\ln\frac{K_i}{F_0}
  -
  \ln^2\left(\frac{K_i}{F_0}\right)
  \right\}
  Q_{K_i}\Delta K_i
  \right)
  + \varepsilon_3
  $$

  $$
  S =
  \frac{
  P_3 - 3P_1P_2 + 2P_1^3
  }{
  \left(P_2 - P_1^2\right)^{3/2}
  }
  $$

- Fair skew swap strike following Lee (2024), where the skew swap strike is replicated using a weighted strip of OTM puts and calls:

  $$
  K_s =
  \frac{2}{T S_0}
  \left(
  \int_{0}^{S_0}
  \operatorname{put}(S_0, K)
  \frac{1}{K^2}
  (S_0 - K)\, dK
  +
  \int_{S_0}^{\infty}
  \operatorname{call}(S_0, K)
  \frac{1}{K^2}
  (S_0 - K)\, dK
  \right)
  $$

- And Kozhan, Neuberger, and Schneider (2012), where $$v^{E}_{t,T}$$ is the fixed leg of an entropy contract, and $$v^{L}_{t,T}$$ is our favorite the variance swap.

  $$
  \operatorname{skew}_{t,T}
  =
  3
  \frac{
  v^{E}_{t,T} - v^{L}_{t,T}
  }{
  \left(v^{L}_{t,T}\right)^{3/2}
  }
  $$

  $$
  v^{L}_{t,T}
  =
  2
  \sum_{K_i \leq F_{t,T}}
  \frac{P_{t,T}(K_i)}{B_{t,T}K_i^2}
  \Delta I(K_i)
  +
  2
  \sum_{K_i > F_{t,T}}
  \frac{C_{t,T}(K_i)}{B_{t,T}K_i^2}
  \Delta I(K_i)
  $$

  $$
  v^{E}_{t,T}
  =
  2
  \sum_{K_i \leq F_{t,T}}
  \frac{P_{t,T}(K_i)}{B_{t,T}K_iF_{t,T}}
  \Delta I(K_i)
  +
  2
  \sum_{K_i > F_{t,T}}
  \frac{C_{t,T}(K_i)}{B_{t,T}K_iF_{t,T}}
  \Delta I(K_i)
  $$


  I know this is a whole lot of math but each formula really is just a different weighting of OTM puts and calls. You can look on my github under py_op to see how I coded these. Below we show how these look plotted over time for estimates using daily S&P 500 options data, where we rebalance the option holdings daily.

![Skew Swap Fixed Legs](/assets/images/skew_swap_strikes-2026-05-25.png)

As you can see the model-free implied skew derived from Ito (2025) and Kozhan, Neuberger, and Schneider (2012) are extremely close so this is a good sign. Now we move onto our realized and implied skew/skewness estimates.

### Realized/Implied Estimates

Below we show rolling metrics for realized and implied spot-vol correlation, where the realized correlation is estimated using historical log returns and ATM IV and the implied correlation is estimated using a calibrated parameter from a stochastic volatility model, or directly taken from the skew curve.

![Spot Vol Correlations](/assets/images/spot_vol_correlations-2026-05-25.png)

It was extremely suprising to me to see how close the implied spot-vol correlation was for each of the metrics. Next we show the same metrics except the covariance version which is easier to compare across implied and realized, and we show the difference in the two which we use to estimate skew risk premium. 

![Spot Vol Covariances](/assets/images/spot_vol_covariance_skew_risk_premium-2026-05-25.png)