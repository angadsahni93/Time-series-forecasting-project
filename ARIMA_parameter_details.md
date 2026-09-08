### What do $p$ and $q$ actually define?

The full $ARMA(p,q)$ equation (on a stationary/differenced series):

$$y_t = c + \underbrace{\phi_1 y_{t-1} + \phi_2 y_{t-2} + \dots + \phi_p y_{t-p}}_{\text{AR part, order } p} + \underbrace{\theta_1 e_{t-1} + \theta_2 e_{t-2} + \dots + \theta_q e_{t-q}}_{\text{MA part, order } q} + e_t$$

- **$p$** = how many *past values* of $y$ are included in the equation ($y_{t-1}$ through $y_{t-p}$).
- **$q$** = how many *past forecast errors* are included ($e_{t-1}$ through $e_{t-q}$).

Each one you add brings its own coefficient ($\phi_i$ or $\theta_i$) that has to be estimated from the data — which is exactly what "parameter" means below.

### What is a "parameter" here?

Each AR term contributes one coefficient $\phi_i$ (multiplying $y_{t-i}$); each MA term contributes one coefficient $\theta_i$ (multiplying $e_{t-i}$). So an $ARMA(p,q)$ model has to *estimate*:

$$k \approx p \text{ (AR coefficients)} + q \text{ (MA coefficients)} + 1 \text{ (constant/intercept)}$$

$k$ is the parameter count. It grows every time you add another lag — e.g. $AR(3)$ has 3 parameters, $MA(3)$ has 3, $ARMA(3,3)$ has 6.

**How AIC/BIC penalize it:**

$$AIC = 2k - 2\ln(L) \qquad BIC = k\ln(n) - 2\ln(L)$$

where $L$ = the fitted model's likelihood (how well it explains the data) and $n$ = number of observations.

- $-2\ln(L)$ rewards a better fit — a model that explains the data well pushes this term down.
- $2k$ (AIC) or $k\ln(n)$ (BIC) is a straight tax on model size: it goes up with $k$ alone, **regardless of what the fitted coefficient values actually are**. A bigger model pays a bigger penalty just for existing.

**Why this stops overfitting:** adding one more AR or MA term almost always improves the fit at least slightly (more flexibility to bend toward the training data), even when that improvement is just fitting noise. AIC/BIC forces that extra term to "earn its keep" — the likelihood has to improve by more than the added $2$ (or $\ln(n)$) penalty, or the simpler model wins. BIC's penalty grows with $n$, so it punishes extra parameters more harshly than AIC as the sample size increases.

**How is $L$ actually computed for a forecasting model?**

ARIMA/ARMA models assume the leftover forecast errors $e_t = y_t - \hat{y}_t$ are independent and normally distributed with mean 0 and some variance $\sigma^2$ — i.e. $e_t \sim N(0, \sigma^2)$. The likelihood of the whole fitted model is just the product of each observation's probability density under that assumption:

$$L = \prod_{t=1}^{n} \frac{1}{\sqrt{2\pi\sigma^2}} \exp\left(-\frac{e_t^2}{2\sigma^2}\right)$$

In practice this product is turned into a **log-likelihood** (sums are easier/more stable than products of small numbers):

$$\ln(L) = -\frac{n}{2}\ln(2\pi\sigma^2) - \frac{1}{2\sigma^2}\sum_{t=1}^{n} e_t^2$$

**What every symbol means:**

| Symbol | Meaning |
|---|---|
| $n$ | number of observations (data points) in the series being fit |
| $t$ | the time index, running from $1$ to $n$ |
| $y_t$ | the actual observed value at time $t$ |
| $\hat{y}_t$ | the model's fitted/predicted value at time $t$ |
| $e_t$ | the residual (forecast error) at time $t$: $e_t = y_t - \hat{y}_t$ |
| $\sigma^2$ | the variance of the residuals — how spread out the errors are |
| $\hat{\sigma}^2$ | the *estimated* $\sigma^2$, computed from the data as $\frac{1}{n}\sum e_t^2$ (mean squared residual) |
| $\pi$ | the constant pi ($\approx 3.14159$) — appears because it's a normal-distribution formula |
| $\sum_{t=1}^{n}$ | "sum over every time point from 1 to $n$" |
| $\prod_{t=1}^{n}$ | "multiply together every time point's term, from 1 to $n$" |
| $\exp(\cdot)$ | the exponential function, $e^{(\cdot)}$ |
| $\ln(\cdot)$ | the natural logarithm |
| $L$ | the likelihood — the probability of observing this exact data, given the fitted model |
| $\ln(L)$ | the log-likelihood — same information as $L$, just on a log scale (easier to compute/compare) |

$\sigma^2$ itself is estimated as the average squared residual, $\hat{\sigma}^2 = \frac{1}{n}\sum e_t^2$. So concretely: fit the model → get its residuals $e_t$ → plug them into the formula above to get $\ln(L)$ → plug that into the AIC/BIC formulas. `statsmodels` does all of this for you automatically — after `.fit()`, the log-likelihood, AIC and BIC are directly available as `model.llf`, `model.aic`, `model.bic`.

### Why is larger $p$/$q$ worse?

It's not that a bigger $p$/$q$ is *inherently* wrong — it's that it costs you in several compounding ways, beyond just the AIC/BIC penalty term itself:

- **Fewer degrees of freedom per parameter:** with a fixed amount of data $n$, every extra coefficient you estimate eats into the data available to pin down all the others — estimates get noisier/less reliable ("overfit" in the statistical-efficiency sense), especially with small $n$ (like our $n \approx 34$ monthly billings series).
- **Fits noise, not signal:** past the point where the ACF/PACF genuinely cuts off, extra lags aren't capturing real momentum or error-correction — they're just fitting random fluctuations specific to *this* sample. That noise won't repeat in future data, so forecasts get worse even though the in-sample fit looks better.
- **Coefficient instability / near-collinearity:** nearby lags of the same series are often correlated with each other (e.g. $y_{t-1}$ and $y_{t-2}$ move together), so cramming in many lags makes individual coefficient estimates unstable and hard to interpret — small changes in the data can swing them a lot.
- **Directly larger AIC/BIC penalty:** as shown above, $k$ grows with $p+q$, so the penalty term ($2k$ or $k\ln(n)$) grows too — a larger model has to earn a correspondingly larger improvement in fit just to break even.

Net effect: larger $p,q$ trades a better *in-sample* fit for a worse *out-of-sample* (real future forecasting) one — which is the opposite of what you actually want.
