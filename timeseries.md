## Time Series analysis

In this section we'll try a first approach to measure the impact of the Spanish confinement in twitter user activity. The data that we have consists of 17.000 timelines of Twitter users. We will calculate some of our measurements with the aggregated data of some users, some of them will be done with several examples and some of them will be computed with each of the timelines in the dataset.

### Seasonality and trends

We'll pick several timelines as example (700) and we will calculate the autocorrelation plot (`acf`), for every one of them and see if overall there is some seasonality.

When we calculate the mean over all `acf`, we can see clearly

would give us a higher value for the weeks. break between seasonality and trends gives us any valuable information. The data

### Linear breakpoints

This method here will be applied to the whole set of timeseries. As we are trying to measure the effect of an intervention (confinement) in the overall set of users, it'll be useful to get a more simplified version of the timeseries, one that just gets the *essential* information, as we are dealing with a lot of timeseries. The function `breakpoints` in `R` package `strucchange` gives us the optimal breakpoints for the timeseries. Internally what this function does is optimizes a piecewise linear fit for each $i$ number of breaks up to $n$ (maximum number of breaks). Although it can be done in python, there is no efficient method to get the same result, and as we are dealing with a lot of timeseries, we'll stick to `R` with the wrapper for python `rpy`. 

The code to 

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



### Examples of ARIMA



 https://datascienceplus.com/arima-models-and-intervention-analysis/

https://www.seanabu.com/2016/03/22/time-series-seasonal-ARIMA-model-in-python/

https://www.analyticsvidhya.com/blog/2016/02/time-series-forecasting-codes-python/