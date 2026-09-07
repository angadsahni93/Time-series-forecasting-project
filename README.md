# Semiconductor sales & NVIDIA stock: Coursera Project

## Project Overview:
Analyse Raw data for Actionable insights.


## Why Semiconductor industry:
Sales are highly cyclic (boom and slowdown phases).
Critical industry for downstream industries (smartphone, cloud computing, etc.)


## Business Questions:
1. How does the semiconductor demand **change** over time?
<br>

   1a. Is the demand **slowing down**?
<br>

2. Are there **seasonal patterns** in demand?
<br>

3. What will be the **expected demand** over the next few months?

*Overarching Question:* How can companies plan **scaling-up or reducing output**, based on the seasonal patterns of demand?


## Approach/Methodology:
```mermaid
flowchart TD
    A[Data collection] --> B[Data cleaning/preprocessing]
    B --> C[Exploratory Data Analysis:<br/>for relevance to business questions]
    C --> D[Time series visualizations]
    D --> E[Trend and Seasonality analysis]
    E --> F[Forecasting model:<br/>industry trends]
```


## Forecasting Model Concepts: AR, MA, ARIMA
**AR** = uses past **values**. Predicts $y_t$ using $y_{t-1}, y_{t-2}, ..., y_{t-p}$ + noise (residuals) — $p$ = number of lagged observations included (AR order). AR captures **momentum**.

**MA** = uses past **forecast errors**. Predicts $y_t$ using noise (residuals, mistakes) from the previous $q$ periods — $q$ = number of lagged forecast errors included (MA order).

![AR forecasts from past values; MA forecasts from past forecast errors](images/ar_ma_forecast.png)

**AR + MA + Integration (differencing) = ARIMA** (AutoRegressive Integrated Moving Average) — combines AR + MA, plus differencing to first make a non-stationary series stationary.

$$ARIMA(p, d, q) \quad p = \text{AR order (lagged values)}, \quad d = \text{differencing order}, \quad q = \text{MA order (lagged forecast errors)}$$

**Example:** $ARIMA(1,1,1)$ — $y_t$ depends on 1 lagged value ($p=1$) and 1 lagged forecast error ($q=1$), fitted on the series after differencing it once ($d=1$) to remove trend.
