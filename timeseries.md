## Time series analysis

In this section we'll try a first approach to measure the impact of the Spanish confinement in twitter user activity. The data that we have consists of 17.000 timelines of Twitter users. We will calculate some of our measurements with the aggregated data of some users, some of them will be done with several examples and some of them will be computed with each of the timelines in the dataset.

###  Seasonality and trends

We'll pick several timelines as example (700) and we will calculate the autocorrelation plot (`acf`), for every one of them and see if overall there is some seasonality.

When we calculate the mean over all `acf`s, we can see spikes at each $k$ multiple of 7. This means a clear tendency towards a weekly frequency period.

<iframe height='320' scrolling='no' src='../tfm-plots/timeseries-mean-acf.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>

Now we are going to explore some examples of the weekly periodic timelines, decomposing them in (additive) trend/seasonal/residual series. We'll try to see if there is any pattern in the profiles whose timelines are being weekly periodic or not, and what are the rises and falls of twitter activity within the period.

- Explore example



### Linear breakpoints

As our goal is to try to measure the effect of an intervention (confinement) in the overall set of users, it'll be useful to get a more simplified version of the timeseries, one that just gets the *essential* information, as we are dealing with a lot of timeseries. This method here will be applied to the whole set of timeseries.  The function `breakpoints` in `R` package `strucchange` gives us the optimal points where to break the time series so there is the least amount of breakpoints possible that describe the best the original time series.  Although it can be done in python, there is no efficient method to get the same result, and as we are dealing with a lot of timeseries, we'll stick to `R` with the wrapper for python `rpy`. 

We'll go through an example of timeline to see how this function internally works. First we pick a random user (`id=1000092194961838080`) and plot its activity timeline:

<figure style="text-align:center">
    <iframe height='340' scrolling='no' src='../tfm-plots/timeseries-example-breakpoints.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.1 - Activity of user 1000092194961838080.</figcaption>
</figure>



The following code creates the breakpoints and retrieves the level of each interval:

```python
from rpy2 import robjects
import rpy2.robjects.packages as rpackages
rpackages.importr('strucchange')
def get_breakpoints_and_levels(id_user):
    model = pickle.loads(open('../data/models/{}'.format(id_user), 'rb').read())
    freq = model['freq']
    formula = robjects.Formula('freq_tweet ~ 1')
    env = formula.environment
    env['freq_tweet'] = robjects.r['ts'](robjects.FloatVector(freq.values),  start=freq.min())
    breakpoints = robjects.r['breakpoints'](formula)
    fitted = robjects.r['fitted'](breakpoints, breaks=len(breakpoints[0]))
    return {freq.index[int(i)]:fitted[int(i)] for i in [0.0]+list(breakpoints[0])}
```

Internally what the function `breakpoints`does is optimizes a piecewise linear fit, for each $i$ number of breaks, up to $n$ (maximum number of breaks), and it compares among the `i`'s which is the best optimized interval. It does this by using Bayesian Information criterion $(BIC)$. $BIC$ is a common criterion to do model selection among finite set of models. It's a way to measure the maximum likelihood of a function with the minimum amount of complexity (without overfitting). We can see the results for this in the figure 2.

<figure style="text-align:center">
    <img src='../tfm-plots/static/timeseries-BIC-breakpoints.png' height=350>
    <figcaption>Fig.2 - BIC for the optimized interval for each number of breakpoints. In the case of the example the best description of the timeseries with the least breakpoints is with 3 breakpoints (4 intervals)</figcaption>
</figure>


Another `R` function that appears in the code is `fitted`, retrieves the level of each interval, also computed by the `breakpoints` function. The final result for each

<figure style="text-align:center">
    <iframe height='340' scrolling='no' src='../tfm-plots/timeseries-example-breakpoints-s.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.3 - Activity of user 1000092194961838080 with breakpoints and levels.</figcaption>
</figure>


As we did this for every profile in the dataset, we'll try to see some general insights. One of the problems that we'll encounter throughout this paper is the what we will call *starting date problem*. This consists in not having a consistent date from which we begin to have the activities of the user activities in our data. This is because we took the timelines not homogenously and also Twitter API gets only the latest 3200 activities from a user, that creates a correlation between the first recorded date of the user and its activity frequency. When we want to extract general conclusions of how a period of time affected the users we need to take that into account. 

[timelines activities viz?]

For this we'll compute the jump value for every timeline, that means for each breakpoint, we'll compute  how much it changed from previous value. To not have frequency biases we'll normalize this value for the mean of the timeline. 
$$
\begin{equation*}
\bar N:= \text{observed mean}\\
\tau_i:= \text{date of the breakpoint}\\
L_i:= \text{level of the interval previous to the breakpoint}\\

J_i:=L_{i+1}-L_i\\
J'_i:=\frac{J_i}{\bar N}
\end{equation*}
$$

In the case of the example:
$$
\begin{equation}
\bar N=3.57\\
\tau_0=\text{08-Nov-19},  J_0=2.43, J'_0=0.68 \\  \tau_1=\text{16-Feb-20},J_1=4.16, J'_1=1.16\\  \tau_2=\text{30-Mar-20},J_2 = -4.86, J'_2=1.37
\end{equation}
$$



### Analysis

- Seasonality -> per period, per type

- Num breakpoints -> per period, per type
- Deviation breakpoints -> per period, per type
- J's
  - J's per type
  - Total J's



<figure style="text-align:center">
    <iframe height='340' scrolling='no' src='../tfm-plots/timeseries-sum-js.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean (J') from all timelines.</figcaption>
</figure>



### ARIMA models [no TFM]

We will fit and test three ARIMA models with the help of the information we gathered so far. 

**Model 1: not seasonal**

**Model 2: seasonal (lag=7)**

As we have seen in the first part of this paper, some of the timelines in the dataset have the seasonality of lag 7 (weekly).

**Model 3: seasonal with shift regressors**





 https://datascienceplus.com/arima-models-and-intervention-analysis/

https://www.seanabu.com/2016/03/22/time-series-seasonal-ARIMA-model-in-python/

https://www.analyticsvidhya.com/blog/2016/02/time-series-forecasting-codes-python/