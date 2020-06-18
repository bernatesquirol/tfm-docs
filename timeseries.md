## Time series analysis

In this section we'll try a first approach to measure the impact of the Spanish confinement in twitter user activity. The data that we have consists of 17.000 timelines of Twitter users. We will calculate some of our measurements with the aggregated data of some users, some of them will be done with several examples and some of them will be computed with each of the timelines in the dataset.

###  Seasonality and trends

We'll pick several timelines as example (700) and we will calculate the autocorrelation plot (`acf`), for every one of them and see if overall there is some seasonality.

When we calculate the mean over all `acf`s, we can see spikes at each $k$ multiple of 7. This means a clear tendency towards a weekly frequency period.

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/timeseries-mean-acf.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.1 - Mean acf for 700 samples.</figcaption>
</figure>

### Linear breakpoints

As our goal is to try to measure the effect of an intervention (confinement) in the overall set of users, it'll be useful to get a more simplified version of the timeseries, one that just gets the *essential* information, as we are dealing with a lot of timeseries. This method here will be applied to the whole set of timeseries.  The function `breakpoints` in `R` package `strucchange` gives us the optimal points where to break the time series so there is the least amount of breakpoints possible that describe the best the original time series.  Although it can be done in python, there is no efficient method to get the same result, and as we are dealing with a lot of timeseries, we'll stick to `R` with the wrapper for python `rpy`. 

We'll go through an example of timeline to see how this function internally works. We'll pick one example for the sake of explainability, with the Twitter activity plotted in $\text{Fig.1}$.

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

Internally what the function `breakpoints`does is optimizes a piecewise linear fit, for each $i$ number of breaks, up to $n$ (maximum number of breaks), and it compares among the `i`'s which is the best optimized interval. It does this by using Bayesian Information criterion $(BIC)$. $BIC$ is a common criterion to do model selection among finite set of models. It's a way to measure the maximum likelihood of a function with the minimum amount of complexity (without overfitting). We can see the results for this in the figure 2. We'll stablish the maximum number of breaks in $n=5$.

<figure style="text-align:center">
    <img src='../tfm-plots/static/timeseries-BIC-breakpoints.png' height=350>
    <figcaption>Fig.2 - BIC for the optimized interval for each number of breakpoints. In the case of the example the best description of the timeseries with the least breakpoints is with 3 breakpoints (4 intervals)</figcaption>
</figure>


Another `R` function that appears in the code is `fitted`, retrieves the level of each interval, also computed by the `breakpoints` function. The final result for each

<figure style="text-align:center">
    <iframe height='340' scrolling='no' src='../tfm-plots/timeseries-example-breakpoints-s.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.3 - Activity of user 1000092194961838080 with breakpoints and levels.</figcaption>
</figure>


As we did this for every profile in the dataset, we'll try to see some general insights. One of the problems that we'll encounter throughout this paper is the what we will call *initial date problem*. This consists in not having a consistent date from which we begin to have the activities of the user activities in our data. This is because we took the timelines not homogenously and also Twitter API gets only the latest 3200 activities from a user, that creates a correlation between the first recorded date of the user and its activity frequency. When we want to extract general conclusions of how a period of time affected the users we need to take that into account. 

[timelines activities viz?]

For this we'll compute the jump value for every timeline, that means for each breakpoint, we'll compute  how much it changed from previous value. To not have frequency biases we'll normalize this value for the mean of the timeline. 
$$
\begin{equation*}
\bar N:= \text{observed mean}\\
\tau_0:=\text{starting point}\\
\tau_i:= \text{date of the breakpoint}\\
L_i:= \text{mean of the interval }[\tau_i,\tau_{i+1})\\
J_i:=L_{i}-L_{i-1}\\
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

With the label **`levels_dict`** we'll add a dictionary with keys $\tau_i$ and values $L_i$ as as a feature in our users database.

### Analysis

In this section we'll analyse certain aspects of the features we have computed so far.

#### Seasonality

We have computed the `seasonal_decompose` of each timeline in our database, that means decomposing the activity in (additive) trend/seasonal/residual series. The plot in $\text{Fig.4}$ we can see the differences between user types. We have a lot less politicians than any other group, this why is so different than the other types.

<figure style="text-align:center">
    <iframe height='420' scrolling='no' src='../tfm-plots/timeseries-seasonal-type.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean (J') from all timelines.</figcaption>
</figure>

#### Breakpoints

- Num breakpoints -> per period, per type

<figure style="text-align:center">
    <iframe height='370' scrolling='no' src='../tfm-plots/timeseries-breakpoints-analysis.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean of the jump between levels (J') from all timelines.</figcaption>
</figure>

#### Levels

<figure style="text-align:center">
    <iframe height='820' scrolling='no' src='../tfm-plots/timeseries-sum-js-types.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean of the jump between levels (J') from all timelines.</figcaption>
</figure>

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/timeseries-sum-js-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean of the jump between levels (J') from all timelines.</figcaption>
</figure>




### ARIMA models

We will fit and test three ARIMA models in our example with the help of the information we gathered so far.  

- Explain ARIMA

**Model 1: autoarima**

```python
model_1 = robjects.r['auto.arima'](env['freq_tweet'], trace=True)
```

```R
Fitting models using approximations to speed things up...
 ARIMA(0,1,0)                    : 1392.701
 ARIMA(0,1,0) with drift         : 1394.732
 ARIMA(0,1,1)                    : 1297.803
 ARIMA(0,1,1) with drift         : 1299.847
 ARIMA(0,1,2)                    : 1296.231
 ARIMA(0,1,2) with drift         : 1298.283
 ARIMA(1,1,0)                    : 1348.734
 ARIMA(1,1,0) with drift         : 1350.78
 ARIMA(1,1,1)                    : 1297.113
 ARIMA(1,1,1) with drift         : 1299.162
 ARIMA(1,1,2)                    : 1299.158
 ARIMA(1,1,2) with drift         : 1301.218
 ARIMA(2,1,0)                    : 1327.977
 ARIMA(2,1,0) with drift         : 1330.04
 ARIMA(2,1,1)                    : 1302.686
 ARIMA(2,1,1) with drift         : 1304.751
 ARIMA(2,1,2)                    : 1300.697
 ARIMA(2,1,2) with drift         : 1302.793

Best model: ARIMA(0,1,2) 
Coefficients:
          ma1      ma2

      -0.6861  -0.1220

s.e.   0.0616   0.0649
sigma^2 estimated as 9.385:  log likelihood=-646.8
AIC=1299.61   AICc=1299.7   BIC=1310.23
```



**Model 2: seasonal (lag=7)**

If we take the weekly seasonality into account, we can see a small improvements in the model.

```python
model_2_1 = robjects.r['Arima'](
    env['freq_tweet'], 
    order = robjects.IntVector([0,1,2]),
    seasonal =  robjects.r['list'](order = robjects.IntVector([0,0,1]), period = 7), 
)
```

```R
Coefficients:
          ma1      ma2    sma1

      -0.6861  -0.1333  0.0900

s.e.   0.0609   0.0660  0.0605
sigma^2 estimated as 9.339:  log likelihood=-645.7
AIC=1299.39   AICc=1299.55   BIC=1313.56
```



```python
model_2_2 = robjects.r['Arima'](
    env['freq_tweet'], 
    order = robjects.IntVector([0,1,2]),
    seasonal =  robjects.r['list'](order = robjects.IntVector([1,0,0]), period = 7), 
)
```

```R
Coefficients:
          ma1      ma2    sar1

      -0.6858  -0.1341  0.0989

s.e.   0.0609   0.0662  0.0632
sigma^2 estimated as 9.331:  log likelihood=-645.58
AIC=1299.16   AICc=1299.32   BIC=1313.32
```



**Model 3: seasonal with shift regressors**

- What are regressors

If we check again the `auto.arima` function, this time with regressors, we get the best

```python
model_4_1 = robjects.r['auto.arima'](env['freq_tweet'], trace=True, xreg=fitted, )
```

```R
Best model: Regression with ARIMA(0,0,1) errors
Coefficients:
         ma1    xreg
      0.1873  0.9999
s.e.  0.0589  0.0510

sigma^2 estimated as 8.258:  log likelihood=-632.5
AIC=1270.99   AICc=1271.09   BIC=1281.63
```

If we add temporality:

```python
model_4_2 =  robjects.r['Arima'](
    env['freq_tweet'], 
    order = robjects.IntVector([0,0,1]), 
    xreg=fitted,
    seasonal =  robjects.r['list'](order = robjects.IntVector([1,0,0]), period = 7),
)
```

```R
Coefficients:
         ma1    sar1  intercept    xreg
      0.1731  0.1685    -0.1312  1.0363
s.e.  0.0596  0.0624     0.4733  0.1136

sigma^2 estimated as 8.086:  log likelihood=-628.89
AIC=1267.78   AICc=1268.02   BIC=1285.5
```

The problem in fitting the Arima model with regressors is that, if we want to predict, we need the regressor as input. If we could have an overall estimate of how 

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/residuals-slider.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean of the jump between levels (J') from all timelines.</figcaption>
</figure>




Time Series Analysis With Applications in R by Jonathan D. Cryer, Kung-Sik Chan

- Explain residua

| k\model | 1        | 2_1      | 2_2      | 3_1          | 3_2      |
| ------: | -------- | -------- | -------- | ------------ | -------- |
|       1 | 0.871256 | 0.676499 | 0.905459 | 0.704804     | 0.803445 |
|       2 | 0.985991 | 0.913195 | 0.975867 | 0.770529     | 0.883243 |
|       3 | 0.986286 | 0.942512 | 0.979900 | 0.491782     | 0.547569 |
|       4 | 0.997578 | 0.982539 | 0.994038 | 0.426820     | 0.454997 |
|       5 | 0.986176 | 0.979301 | 0.992030 | 0.520453     | 0.552379 |
|       6 | 0.801770 | 0.992874 | 0.997364 | **0.087242** | 0.666250 |
|       7 | 0.879470 | 0.997623 | 0.999127 | **0.092944** | 0.600149 |
|       8 | 0.841681 | 0.988918 | 0.994982 | 0.135140     | 0.693404 |
|       9 | 0.855799 | 0.990754 | 0.995835 | 0.189310     | 0.770193 |
|      10 | 0.903568 | 0.995453 | 0.998018 | 0.252571     | 0.832601 |

 https://datascienceplus.com/arima-models-and-intervention-analysis/

https://www.seanabu.com/2016/03/22/time-series-seasonal-ARIMA-model-in-python/

https://www.analyticsvidhya.com/blog/2016/02/time-series-forecasting-codes-python/