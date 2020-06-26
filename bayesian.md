# Bayesian model

In this section we'll try a different approach to measure the impact of the Spanish confinement in twitter user activity. The data that we have consists of 17.000 timelines of Twitter users. A timeline is the collection of the most recent tweets published by a user. Our goal is to simplify the activity data of the user so we can measure the most important changes in all users behaviour and the impact of confinement and COVID19 in Twitter. We aggregate our data by days.

## Prior

In order to measure the impact of the Spanish confinement in Twitter, we will analyse multiple users timelines and we'll fit a *Bayesian switchpoint* model for each timeline. This is a very basic model that assumes that there is one day ($\tau$) the user changes its behaviour, either increasing the frequency of tweet or decreasing it. The fact that we have just one switchpoint will be great for our analysis, as we have a huge collection of timelines, and we want to extract a *minimalist* model.

We'll call $T_{i}$ to the expected number tweets for the day $i$ created by a given user. Although there is evidence that under certain circumstances human interaction follows a non-Poisson process ($\text{Pareto}$, $\text{Weibull}$, $\text{Log-Normal}$), there is a wide range of social interaction models that assume that the communications among individuals are randomly distributed in time, and thus Poissonian. [[1]](./#ref1)

Here $T_i$ will be modelled following a $\text{Poisson}$ distribution and at day $\tau$ the parameter of the distribution will change from $\lambda_1$ to $\lambda_2$.  The $\text{Poisson}$ distribution is defined as:
$$
P(Z = k) =\frac{ \lambda^k e^{-\lambda} }{k!}, \; \; k=0,1,2, \dots
$$
The parameter $\lambda$ must be positive, and if we increase $\lambda$ we add more probability to large $k$'s, and if we decrease it, small $k$ will have more probability. One important fact about the $\text{Poisson}$ distribution is that the expected value of the distribution $E(\text{Poisson}(\lambda)|\lambda)=\lambda$.

These two $\lambda$'s will modelled following $\text{Exponential}$ distributions. The $\text{Exponential}$ distribution always gives a positive number, and is continuous like the parameter in the $\text{Poisson}$ and has a really straight forward expected value. Is a continuous defined by:
$$
f_Z(z | \alpha) = \alpha e^{-\alpha z }, \;\; z\ge 0
$$
And it's expected value is $E(\text{Exponential}(\alpha)|\alpha)=\frac{1}{\alpha}$.

So the prior for our model is, with $n=\text{#days}$:
$$
\begin{align*}\lambda_{1}^{(0)} &\sim \text{Exponential}(\text{rate}=\alpha) \\
\lambda_{2}^{(0)} &\sim \text{Exponential}(\text{rate}=\alpha) \\
\tau &\sim \text{Uniform}[\text{low}=0,\text{high}=1) \\
\text{for }  i &= 1\ldots n: \\
\lambda_i &= \begin{cases} \lambda_{1}^{(0)}, & \tau \leq i/n \\ \lambda_{2}^{(0)}, &   \tau  > i/n\end{cases}\\
 T_i &\sim \text{Poisson}(\text{rate}=\lambda_i)
 \end{align*}
$$
We only have one hyperparameter in this Bayesian graph that is $\alpha$. From the expected values of the distribution we have:
$$
E(T_i)=E(\text{Poisson}(\lambda_i)|\lambda_i)=E(\lambda_i)=E(\text{Exponential}(\alpha)|\alpha)=\frac{1}{\alpha}
$$
And as we want $E(T_i)=\bar{X}$, we'll have our initial rate for the $\lambda_i$'s be: $\alpha=\frac{1}{\overline{X}}$.

With this steps we converted our prior in an informative prior. We could have also created two $\alpha_i$, one for every $\lambda_i$, but the values of those $\alpha_i$ wouldn't be so clear and unbiased. For example we could have initialized $\alpha_0$ to the inverse of the mean of the first days, and $\alpha_1$ to the inverse of the mean of the last days.

This model will fit a $\tau$ that will be the day of the _distribution change_. Our prior for this $\tau$ will be a uniform, as we want all days to have the same probability to be the switch point. We expect to see a the change of behaviour near the confinement dates. [[2]](./#ref2)

## Model fit

The implementation is done with [Tensorflow probability](https://www.tensorflow.org/probability/) (TFP) in Python 3.6. To fit the data into the model and get our posterior we'll use Markov Chain Monte Carlo algorithm. This algorithm what it does is finding the most probable parameters for a given prior by sampling from the prior and fitting the data. When we specify the priors, we create a n-dimensional surface that can be modified in certain ways, depending on the parameters. The MCMC algorithm travels the space of all possible surfaces and finds which have the higher probability to have generated our data. There are several types of MCMC algorithm that change how the optimization works, we'll use Hamiltonian Monte Carlo. We need to create our prior in the TFP language so the `tfp.mcmc.HamiltonianMonteCarlo` function can optimize it:

```python
def joint_log_prob(count_data, lambda_1, lambda_2, tau):
    tfd = tfp.distributions
    # alpha is the initial value of both exponentials
    alpha = (1. / tf.reduce_mean(count_data))
    # exponential lambda
    rv_lambda_1 = tfd.Exponential(rate=alpha)
    rv_lambda_2 = tfd.Exponential(rate=alpha)
	# uniform tau
    rv_tau = tfd.Uniform()
	# switch
    lambda_ = tf.gather([lambda_1, lambda_2],indices=tf.cast(tau * tf.cast(tf.size(count_data), dtype=tf.float32) <= tf.cast(tf.range(tf.size(count_data)), dtype=tf.float32), dtype=tf.int32))
    # T_i
    rv_observation = tfd.Poisson(rate=lambda_)
	# return joint log probability of the model
    return (
         rv_lambda_1.log_prob(lambda_1)
         + rv_lambda_2.log_prob(lambda_2)
         + rv_tau.log_prob(tau)
         + tf.reduce_sum(rv_observation.log_prob(count_data))
    )
```
After this, we need to create the kernel, where tensorflow optimizes the function. The kernel will output some results about the convergence of the algorithm and the samples the algorithm found to $\lambda_i$ and $\tau$.

## Model test

We'll fit a series of artificial data, some of which will follow the prior, and some of which won't, to see how well the model behaves. In order to create artificial timelines, we'll create a key valued dictionary $\tau_i$ and $L_i$, that will be similar to breakpoints and levels dictionary we created in the [linear breakpoints](./timeseries.html#linear-breakpoints) section in the timeseries study. The dictionary has as keys ($\tau_i$) the dates of the breakpoints and as values ($\lambda_i=L_i$) the value of the expected tweet activity for the following period. The first key is always the beginning of the timeseries.

We'll use `pvalue` to get a notion of model fitness to the real data. We used an adaptation of the frequentist Kolmogorov-Smirnov test.  We compute different artificial datasets based on the output of the model, and average the `pvalue` of the KS-test between those artificial datasets and the real data (in the case of the following examples is also artificial). If the `pvalue` is really low the

### Single switch

- $\text{E1}$ : `{2019-09-12: 3, 2020-03-09: 4}`, `pvalue=0.56`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/bayesian-test-3-4.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.1 - Change from 3 to 4 at 10th Mar.</figcaption>
</figure>

- $\text{E2:}$ `{2019-09-12: 3, 2020-03-09: 7}`, `pvalue=0.77`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/bayesian-test-3-7.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.2 - Change λ from 3 to 7 at 10th Mar.</figcaption>
</figure>

- $\text{E3}$: `{2019-09-12: 3, 2020-01-22: 7}`, `pvalue=0.82`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/bayesian-test-3-7-jan.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.2 - Change λ from 3 to 7 at 10th Mar.</figcaption>
</figure>

We can see two immediate observations from the previous plots:

- We will have clearer distributions for $\tau$ when the change in the $\lambda_i$'s is large, as we expect $\tau$ to be only one day. The more unclear $\tau$ is, the wider will be $\lambda_2$ distribution.
- We will have a thinner distribution in $\lambda_i$ when the change happens in the middle. Else we'll have more uncertainty (as we have less data) in one of them.

### Multiple switches

The real data may have more than one switch, but our model assumes there is just one. 

- $\text{E4}$: `{2019-09-12: 1, 2019-01-12: 5, 2020-03-02: 2}`, `pvalue=0.46`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/bayesian-test-3-7-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.2 - Change λ from 3 to 7 at 10th Mar.</figcaption>
</figure>

- $\text{E5}$: `{2019-09-12: 1, 2019-10-12: 5, 2020-03-02: 2}`, `pvalue=0.50`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/bayesian-test-3-7-2-bis.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.3 - Change λ from 3 to 7 at 10th Mar.</figcaption>
</figure>

- $\text{E6}$: `{2019-09-12: 1, 2019-11-13: 5, 2020-01-13: 1, 2020-03-14: 5}`, `pvalue=0.01`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/bayesian-test-1-5-1-5.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Change λ from 1 to 5 at 13th Oct and 14 Apr.</figcaption>
</figure>
In the previous examples we can see how in case of two changes, or three changes, the model will get only one of them. We can see how the `pvalue` is way lower in $\text{Fig.4}$. We can conclude we can use the `pvalue` as a way of knowing if there are some existing breakpoints in the data that we missed in the model. When the `pvalue` is low doesn't mean that the model is wrong, the model will have choosen one of the breakpoints that the data will have, but probably other breakpoints may exist.

### Real data

For every timeline in the database, we fitted the model previously mentioned. Let's explore some real timelines and see how the model perfoms.

- `user_id=455018574, pvalue=0.24`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/real-case-1.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.5 - Real example.</figcaption>
</figure>

- `user_id=112894609, pvalue=0.0036`

<figure style="text-align:center">
    <iframe height='425' scrolling='no' src='../tfm-plots/multi-lambda.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.6 - Multi lambda example.</figcaption>
</figure>

The $\text{Fig.6}$ shows something new. If we look at both distributions of $\lambda_i$ they appear to have a binormal distribution. If we look at the $\tau$ we can also see two bumps, one in February 22nd and another in March. What is happening here, is that there are two switchpoints, the rising in February and the decreasing of activity in March. We can see how the rising in March corresponds probably to the part of $\lambda_2 \sim 7.5$ (as $\lambda_1$ needs to be lower that $\lambda_2$) and after March, we would have a $\lambda_3\sim5.6$, that is hidden in $\lambda_2$. This will happen whenever the $\lambda_i$ are near and there are two switches. 

To test the hypotesis that another breakpoint would create a better model, based on the dictionary: `{2020-02-22:6.4, 2020-02-22:7.5, 2020-03-22:5.4}`, with a second breakpoint ($\tau_2$), and a $\lambda_3$, taking into account the bimodality of $\tau$ and $\lambda_i$. We adapted the `pvalue` test to create multiple timelines from that dictionary, and we found a higher `pvalue=0.015`, while the model with only one switchpoint has a `pvalue=0.0036`.

## Other Models

We also analysed other models for a given example. We couldn't run these models in the whole dataset, as it would take too long.

**Sigmoid**

We'll present another model: the sigmoid. instead of the sudden break we had at day $\tau$, we will have a smooth transition between $\lambda$s. With the theorical formulation of the sigmoid prior being:
$$
\begin{align*}
\lambda_{1}^{(0)} &\sim \text{Exponential}(\text{rate}=\alpha) \\
\lambda_{2}^{(0)} &\sim \text{Exponential}(\text{rate}=\alpha) \\
\tau &\sim \text{Uniform}[\text{low}=0,\text{high}=1) \\
\text{for }  i &= 1\ldots n: \\
\lambda_i &= \lambda_{1}^{(0)}+\frac{1}{1+\exp (\frac{i}{n}-\tau)}(\lambda_{2}^{(0)}-\lambda_{1}^{(0)}) \\
 T_i &\sim \text{Poisson}(\text{rate}=\lambda_i)
\end{align*}
$$
In our data this model can remove a bit of uncertainty in the $\tau$, as we don't need a sudden break in the data.

<figure style="text-align:center">
    <iframe height='620' scrolling='no' src='../tfm-plots/bayesian-switch-sigmoid.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.7 - Multi lambda example.</figcaption>
</figure>
**Multilevel model**

Another model we could try, similar to what we did in [breakpoints section]['breakpoints'] is allowing the model to have more than two levels. The difference here is we set up a maximum number of levels ($L_{\text{max}}$) the model can have but no maximum amount of breakpoints between those levels.
$$
\begin{aligned}
\lambda_{0} & \sim \text { Categorical }\left(\left\{\frac{1}{L_{\text{max}}},...,\frac{1}{L_{\text{max}}}\right\}\in v^{L_{\text{max}}}\right) \\
\text{for }  i &= 1\ldots n: \\
&\lambda_{i} \mid \lambda_{i-1} \sim \text { Categorical }\left(\left\{\begin{array}{cc}
p & \text { if } \lambda_{i}=\lambda_{i-1} \\
\frac{1-p}{L_{\text{max}}-1} & \text { otherwise }
\end{array}\right\}\right)   \\
& T_i \sim \text{Poisson}(\text{rate}=\lambda_i)
\end{aligned}
$$

As it happened with the breakpoints model, we'll have to run it with several $L_{\text{max}}$, creating different models. We can see that similarly to what happened with the breakpoints, at 4-level model we reach the optimal amount of information with the minimum amount of levels.

<figure style="text-align:center">
    <img src='../tfm-plots/static/bayesian-multilevel.png' height=450>
    <figcaption>Fig.8 - </figcaption>
</figure>

This model has the advantage that it fits the data better, but the computation time and the simplicity of the result are worse than the linear breakpoints model. For example we can see that the 4-state has a small spike at day $35$, this is useful to analyse one day behaviour, and we wouldn't pick this one with the linear breakpoints model, but when we look at changes that remain through time, the linear breakpoints model probably introduces less noise.

## Analysis

In order to analyse the global behaviour of the users through time, we created a similar visualization of that one done in the timeseries section. We pick for every user the mode of their model parameters $\tilde{\lambda_1}, \tilde{\lambda_2}$, and compute the jump value, in the breakpoint $J=\tilde{\lambda}_{dif}=\tilde{\lambda_2}-\tilde{\lambda_1}$. This assumes a unimodal distribution of $\lambda$ that we already have seen that is not always the case. Then we divide the jump value with the mean of the timeline (as $\lambda\sim L$ is the expected value after the breakpoint): $J'=\frac{J}{\bar T}$.
$$
\begin{equation*}
\bar T:= \text{observed mean}\\
\tau:= \text{distribution of the breakpoint}\\
L_1:= \tilde{\lambda_1}\\
L_2:= \tilde{\lambda_2}\\
J:=L_{2}-L_{1}\\
J':=\frac{J}{\bar T}
\end{equation*}
$$
Similar to what we have done in the breakpoints section, we plot two different series, one for the positive values of $J'$ and the other for the negative values. In $\text{Fig. 9}$ we see plotted the resulting timeline of $\tau*\tilde{\lambda}_{dif}$ for every type of user and in $\text{Fig. 10}$ the overall timeline for all the users.

<figure style="text-align:center">
    <iframe height='820' scrolling='no' src='../tfm-plots/bayesian-type-changes-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.9 - Multi lambda example.</figcaption>
</figure>

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/bayesian-global-changes-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.10 - Multi lambda example.</figcaption>
</figure>



## References

<a name="ref1">**[ 1 ]**</a>: Tianyang Zhang, Peng Cui, Chaoming Song, Wenwu Zhu, Shiqiang Yang. *A Multiscale Survival Process for Modeling Human Activity Patterns*. https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0151473

<a name="ref2">**[ 2 ]**</a>: Cameron Davidson-Pilon. *Bayesian Methods for Hackers. Chapter1. Inferring behavious from text message data*. ISBN: 978-0133902839. https://nbviewer.jupyter.org/github/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers/blob/master/Chapter1_Introduction/Ch1_Introduction_PyMC3.ipynb#Example:-Inferring-behaviour-from-text-message-data

<a name="ref3">**[ 3 ]**</a>: Tensorflow Probability Team. *Bayesian Switchpoint Analysis*. https://www.tensorflow.org/probability/examples/Bayesian_Switchpoint_Analysis

<a name="ref4">**[ 4 ]**</a>: Tensorflow Probability Team. *Multiple changepoint detection and Bayesian model selection*. https://www.tensorflow.org/probability/examples/Multiple_changepoint_detection_and_Bayesian_model_selection

