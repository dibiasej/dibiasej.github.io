---
layout: post
title: "Volatility Surface Risk Premiums"
date: 2026-05-24
---

## Intro

Hi all, this is my first shot at writing, and hopefully something I’ll continue doing. I am by no means a seasoned professional, and I am still very much in the learning process. So, if you read anything in my posts that you disagree with, or think is blatantly wrong, please feel free to reach out and share your thoughts. Also, if you just want to connect, my DMs on LinkedIn and X are always open.

All of the code and charts used in this post can be found on my GitHub. For the results related to this post, you can check out the Volatility-Surface-Risk-Premiums repository, and for the backend calculations, you can look at py_op.

Now onto the good stuff: we’re going to look at risk premiums related to volatility surfaces.

## Premiums
In investing and trading, a risk premium is the compensation investors receive for taking on a specific type of risk. Many risk premiums are well known and have persisted historically. One of my favorite definitions comes from author and volatility expert Euan Sinclair, who describes a risk premium as picking up a $20 bill in the middle of a highway. Most people know the bill is there, but many are not willing to take the risk of walking into traffic to pick it up. Those who do take that risk can be compensated, as long as they don't get run over.

In general there are four things that need to occur in order for something to be a risk premium. 

1. It needs to persist over time, 

2. It needs to occur over many asset classes.

3. It needs to be scalable in size

4. There needs to be a economic rationale to support it. 

The most famous risk premium is probably the equity risk premium which is the reward investors get for buying stocks rather than parking their money in a risk free asset or bank account.

There are numerous risk premiums related to the volatility surface. Broadly speaking, if we can estimate a moment of the risk-neutral distribution implied by option prices and compare it with the corresponding moment of the physical, or realized, distribution, then we can define a potential risk premium around that difference. Practicioners will often estimate a risk premium by constructing a trading strategy and repeatedly trade it over a period of time. In this post, I want to discuss the risk premiums related to the first four moments of the return distribution and some of the common metrics used to quantify them.

These are the variance risk premium (vrp), skew risk premium, skewness risk premium and vol of vol risk premium (vvrp). Many different sources will interchange skew and skewness and some consider them seperate things so it is hard to pin down a canonical definition for them. In this post I consider skewness to be related to the shape of a distribution and skew related to the slope of the implied volatility curve or the spot-vol covariance (although this may be wrong and subject to debate so please share with me your optinion on this).

## Variance Risk Premium

In the world of options and volatility, this is by far the most well known risk premium. It is typically measured as the difference between implied volatility and realized volatility. For the realized component, we can estimate volatility using the sum of squared log returns. For the implied component, I prefer using the fixed leg of a variance swap with the corresponding maturity, which can be approximated using a strip of OTM puts and calls.

There are, however, some caveats to these estimation methods. For example, realized volatility can be estimated in many different ways, and those statistical approaches are widely discussed, so I will not go into them here. On the implied side, we could also use ATM implied volatility as a simpler proxy for the risk premium, although the variance swap estimate is generally a more complete measure because it incorporates information from across the volatility surface.

In the below images we have the 21 day rolling realized volatility and three metrics for our implied part, a rolling iv, the rolling fixed leg of a variance swap and the instantaneous vol calculated from the GVV model (more on this in a future post). These are all calculated from a linear interpolation of the 30 dte closest maturity options.

![Realized Volatility](/assets/images/SPY-Realized-Implied-Volatility-Risk-Premium-Post-2026-05-24.png)

We then find the rolling volatility risk premium using the difference between the fixed leg variance swap strike, and the rolling realized volatility. Note we shift back the realized volatility back to match the date projected by the implied. 

![Volatility Risk Premium](/assets/images/SPY-Variance-Risk-Premium-Post-2026-05-24.png)

I wont talk any more about the vrp because it is widely talked about and researched and I doubt there is much more value to be added. It is important to note that investors/traders usually harvest the variance risk premium using delta hedged straddles.

## Skew/Skewness Risk Premium

If the variance risk premium measures the premium related to dispersion (statistically speaking) then the skewness/skew risk premium measures the premium related to asymmetry. Unlike the vrp there is no agreed upon metric used to quantify this and cannot be measured directly because the conditional physical density is not observable. In this section I will present, in my opinion, a few decent metrics we can use to estimate the skew/skewness risk premium.

Just like the vrp we define the premia as the difference between expected skewness/skew under the risk-neutral measure (implied) and expected skewness/skew under the physical measure (realized). I quantify this in three ways, following the skew-swap framework of Kozhan, Neuberger, and Schneider (2012), the model-free VRP/SRP approach in Ito (2025), and taking the difference between implied - realized spot-vol covariance. Under these frameworks the risk premium can be interpreted as the expected payoff of a skew swap. Most traders capture the premia using option structures like delta hedged risk reversals or some weighting of short OTM puts and long OTM calls.

We will use the following metrics as our estimates for implied skew, realized skew, implied skewness and realized skewness:

###### Implied Skew: 
- Implied spot-vol covaraince: found from an ATM slope of a implied skew curve. I approximate this using calibrated implied parameters from SABR, GVV or some other stochastic volatility model.
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


These formulas are a lot simpler than they look. Each one is really just a different weighting of OTM puts and calls. You can find the code implementation on my GitHub under py_op. Below, I show how these estimates look when plotted over time using daily S&P 500 options data, where the option holdings are rebalanced daily.

![Skew Swap Fixed Legs](/assets/images/skew_swap_strikes-2026-05-25.png)

As you can see the model-free implied skew derived from Ito (2025) and Kozhan, Neuberger, and Schneider (2012) are extremely close so this is a good sign. Now we move onto our realized and implied skew/skewness estimates.

### Realized/Implied Estimates

Below we show rolling metrics for realized and implied spot-vol correlation, where the realized correlation is estimated using historical log returns and ATM IV and the implied correlation is estimated using a calibrated parameter from a stochastic volatility model, or directly taken from the skew curve.

![Spot Vol Correlations](/assets/images/spot_vol_correlations-2026-05-25.png)

Notice how closely the different implied spot-vol correlation measures track each other. The GVV estimate, SABR estimate, and the 90%-110% moneyness skew approximation all show similar dynamics, suggesting that each method is capturing a similar implied spot-vol relationship

Next, I show our three different verions of estimating the skew/skewness risk premium as defined above. The first verison uses differences in realized and implied spot-vol covariance, the second follows Ito (2025) and uses the difference between the model free implied skew and third moment of historical log returns, and the last follows Kozhan, Neuberger, and Schneider (2012)

![Spot Vol Covariances](/assets/images/spot_vol_covariance_skew_risk_premium-2026-05-25.png)

![Akio Ito Skew Risk Premium](/assets/images/akio_implied_realized_skewness_risk_premium-2026-05-25.png)

![Neuberger Skew Risk Premium](/assets/images/neuberger_implied_realized_skew_risk_premium-2026-05-25.png)

For the realized third moment estimate I use close to close prices which may be causing a high variance in the estimate, so while generally it is correct, you can increase the estimation accuracy if you use intraday prices.

Of all the volatility surface related premia discussed in this post, the skew, or skewness, risk premium has the least consensus around how it should be estimated. Hopefully, this section outlined a few useful metrics that you can apply in your own research. Now, we move on to the final premium I want to discuss: the volatility-of-volatility risk premium.

## Volatility of Volatility Risk Premium (VVRP)

Similar to the skew risk premium, there is no single consensus metric for estimating the vol-of-vol risk premium. However, the realized and implied components of VVRP are usually easier to define. Implied vol-of-vol can be estimated from the calibrated parameters of a stochastic volatility model, or from option-implied measures such as VVIX. Realized vol-of-vol can then be estimated using rolling historical measures, such as the volatility of realized volatility, changes in implied volatility, or variance-swap levels. Traders can harvest the vol-of-vol risk premia by trading delta-hedged butterflys or other related option structures.

Here I provide three metrics for the implied vol-of-vol and three for the realized.

###### Implied Vol-of-Vol:
- GVV estimated implied vol-of-vol parameter
- SABR estimated implied vol-of-vol parameter
- VVIX Index 

###### Realized Vol-of-Vol
- Rolling 21 day volatility of Implied volatility
- Rolling 21 day volatility of a Variance Swap
- Rolling 21 day volatility of the close-to-close volatility estimate

These rolling realized metrics are calculated in a similar way to close-to-close realized volatility. Just like with skew, there are many possible approaches, so I only wanted to outline a few in this post. One thing to keep in mind is that the SABR model includes a local-volatility/CEV component, which can affect how the implied parameters should be scaled. This may help explain why the SABR vol-of-vol estimate appears much higher than both the VVIX and GVV vol-of-vol estimates shown below.

I want to add that there is a good paper by the Fed  about estimating the vol-of-vol risk premium where they use the difference between the current level of the VVIX estimate minus the rolling 21 day front month maturity VIX future, Park, Y.-H. (2013). *Volatility of Volatility and Tail Risk Premiums*.

![Vol of Vol](/assets/images/implied_realized_vol_of_vol-2026-025-25.png)

## Conclusion

Overall, the volatility surface contains a lot more information than just the market’s expectation of future volatility. By comparing option-implied estimates with realized or physical estimates, we can begin to isolate different types of risk premiums embedded in the surface. The variance risk premium captures the compensation investors demand for bearing volatility risk, the skew or skewness risk premium gives insight into how the market prices asymmetry and downside risk, and the volatility-of-volatility risk premium reflects the market’s pricing of uncertainty around volatility itself.

None of these estimates are perfect. Each depends heavily on the methodology used, the maturity chosen, the option data available, and the realized metric used for comparison. Still, I think these measures are useful because they give us a structured way to think about what the options market is pricing beyond simple directional views.

In future posts I will discuss risk premiums related dispersion/correlation and the rough volatility risk premium which relates to market implied roughness of the volatiulity path versus what is actually realized. Hopefully this post provided a useful information for thinking about volatility-surface-related risk premiums and some of the ways they can be estimated. As always, all of the code and charts used in this post are available on my GitHub, and I am always open to feedback, corrections, or ideas for improving the analysis.

## References

- Hull, B., & Sinclair, E. (2021). *The Risk-Reversal Premium*. SSRN working paper.

- Ito, A. (n.d.). *Variance Risk Premium, Skewness Risk Premium and Equity Expected Returns*. SSRN preprint.

- Kozhan, R., Neuberger, A., & Schneider, P. (2012). *The Skew Risk Premium in the Equity Index Market*. SSRN working paper.

- Lee, P. B. (2024). *Optimal Skew Swap Replication*. AlphaVols / SAAM.

- Park, Y.-H. (2013). *Volatility of Volatility and Tail Risk Premiums*. Finance and Economics Discussion Series, Federal Reserve Board, 2013-54.

- Yuan, J., Liu, D., Chen, C. R., & Ma, M. (2025). *Estimating Volatility-of-Volatility: A Comparative Analysis*. Economics Letters, 250, 112298.