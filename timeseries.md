# Time series analysis

In this section we will try a first approach to measure the impact of the Spanish confinement in twitter user activity. The data that we have consists of **17.000** timelines of Twitter users. We will calculate some of our measurements with the aggregated data of some users, some of them will be done with several examples and some of them will be computed with each of the timelines in the dataset.

##  Seasonality and trends

We will pick several timelines as example (700) and we will calculate the autocorrelation plot (`acf`), for every one of them and see if there is some seasonality overall.

When we calculate the mean over all `acf`s, we can see spikes at each $k$ multiple of 7. This means a clear tendency towards a weekly frequency period.

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/timeseries-mean-acf.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.1 - Mean acf for 700 samples.</figcaption>
</figure>


## Linear breakpoints

As our goal is to try to measure the effect of an intervention (confinement) in the overall set of users, it will be useful to get a more simplified version of the timeseries, one that just gets the *essential* information, as we are dealing with a lot of timeseries. To do that we will not only find the optimal breakpoints to divide the timeseries in, but also the number of breakpoints that makes the model less complex while describing the model correctly. The function `breakpoints` in `R` package `strucchange` gives us exactly that. Although it can be done in python, there is no efficient method to get the same result, and as we are dealing with a lot of timeseries, we will stick to `R` with the wrapper for python `rpy`. 

We will go through an example of timeline to see how this function works internally. We will pick one example for the sake of explainability, with the Twitter activity plotted in $\text{Fig.1}$.

<figure style="text-align:center">
    <iframe height='340' scrolling='no' src='../tfm-plots/timeseries-example-breakpoints.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.2 - Activity of user 1000092194961838080.</figcaption>
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

Internally what the function `breakpoints`does is optimize a piecewise linear fit, for each $i$ number of breakpoints, up to $n$ (maximum number of breaks); and it compares among the *i*'s which has the lowest Bayesian Information criterion $(BIC)$. $BIC$ is a common criterion to do model selection among finite set of models. It is a way to measure the maximum likelihood of a function (goodness of fit to the real data) with the minimum amount of complexity (without overfitting). We can see the results for this in $\text{Fig.} 2$. We will establish the maximum number of breaks in $n=5$.

<figure style="text-align:center">
    <img src='../tfm-plots/static/timeseries-BIC-breakpoints.png' height=350>
    <figcaption>Fig.3 - BIC for the optimized interval for each number of breakpoints. In the case of the example the best description of the timeseries with the least breakpoints is with 3 breakpoints (4 intervals)</figcaption>
</figure>




Another `R` function that appears in the code is `fitted`, retrieves the level of each interval, also computed by the `breakpoints` function. The final result for each

<figure style="text-align:center">
    <iframe height='540' scrolling='no' src='../tfm-plots/timeseries-example-breakpoints-s.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.3 - Activity of user 1000092194961838080 with breakpoints and levels.</figcaption>
</figure>




As we did this for every profile in the dataset, we will try to see some general insights. In order to do this we will compute the jump value for every timeline, that means for each breakpoint, how much the tweet frequency of the user changed from the previous value. Normalize this jump value for the mean frequency of the timeline, will help avoiding frequency biases in the overall picture. 

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

With the label **`levels_dict`** we will add a dictionary with keys $\tau_i$ and values $L_i$ as a feature in our users database.

## Analysis

In this section we will analyse certain aspects of the features we have computed so far.

### Seasonality

We have computed the `seasonal_decompose` of each timeline in our database, that means decomposing the activity in (additive) trend/seasonal/residual series. The plot in $\text{Fig.4}$ we can see the differences between user types. We can see a Tuesday-Friday activity cycle in politicians that align well with typical political weekly cycles.

<figure style="text-align:center">
    <iframe height='420' scrolling='no' src='../tfm-plots/timeseries-seasonal-type.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Sum of changes relative to the mean (J') from all timelines.</figcaption>
</figure>

### Breakpoints

Here we plot the number of breakpoints for each user type. We can see that it is quite similar among user types. We can also see a trend towards 2-3 breaks and not more, and few users reach the 5 breakpoints. Consequently we can say our maximum number of breakpoints was good enough.

<figure style="text-align:center">
    <iframe height='370' scrolling='no' src='../tfm-plots/timeseries-breakpoints-analysis.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.5 - Number of breakpoints for each user type</figcaption>
</figure>

### Levels

We can see here the sum of $J'$ for all users in our dataset. This values give us the first way to measure the change in user behaviour to COVID19. The green lines represent users changing its activity to a higher level, and red ones to a lower level.

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/timeseries-sum-js-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.6 - Sum of changes relative to the mean of the jump between levels (J') from all timelines.</figcaption>
</figure>

<figure style="text-align:center">
    <iframe height='820' scrolling='no' src='../tfm-plots/timeseries-sum-js-types-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.7 - Sum of changes relative to the mean of the jump between levels (J') from all timelines.</figcaption>
</figure>


## References

<a name="ref1">**[ 1 ]**</a>: Britt, Brian. (2015). *Stepwise Segmented Regression Analysis: An Iterative Statistical Algorithm to Detect and Quantify Evolutionary and Revolutionary Transformations in Longitudinal Data*. 125-144. 10.1007/978-3-319-18552-1_7. https://www.researchgate.net/publication/285613712_Stepwise_Segmented_Regression_Analysis_An_Iterative_Statistical_Algorithm_to_Detect_and_Quantify_Evolutionary_and_Revolutionary_Transformations_in_Longitudinal_Data 

<a name="ref2">**[ 2 ]**</a>: Giorgio Garziano. (2017). *ARIMA models and Intervention Analysis*. https://www.r-bloggers.com/arima-models-and-intervention-analysis/

<a name="ref3">**[ 3 ]**</a>: Cryer, Jonathan &amp; Chan, K.-S. (2008). *Time Series Analysis: With Applications in R*. 10.1007/978-0-387-75959-3. https://www.springer.com/us/book/9780387759586 



