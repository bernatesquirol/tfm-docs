# Ego network analysis

The ego networks are used in social network science to describe the *relational identity* of the individual [[1]](#ref1). There is social cognitive science research that concludes that the self can be modelled in three components: *individual identity*, aspects of the self that are unique, *relational identity*, aspects of the self associated with other relevant individuals for the ego in the network, and *social identity*, definition of the self derived from one's membership in social groups.

We present here a way to construct an ego network for Twitter users, we analyse different aspects of these ego networks, as well as the network evolution through the COVID19 pandemic.

## Concepts

Given a user $u$, let's define:

- $O(u)$: Others of $u$. The set of users that receive 2 or more retweets from $u$.
- $R_u(o)$: Number of retweets that receives a users $o\in O(u)$ from $u$
- $P(R_u)=P(\{ R_u(o), \forall o \in O(u)\})$: Empiric distribution of $R_u(o)$ for all $o\in O(u)$.
- $O_s(u)=\{o\in O(u)| R_u(o)\in \text{Outliers}(P)\}\sub O(u)$: Significative others of $u$. The subset of users for which its number of retweets from $u$ represents an outlier of the $P(R_u)$ distribution

<figure style="text-align:center">
    <iframe frameborder="0" style="width:100%;height:400px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=EgoNetwork.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D13_1OP3DQnsCCYWEf-_Nvcd80MBNHQ6kb%26export%3Ddownload"></iframe>
<figcaption>Fig.1 - Significative other definition.</figcaption>
</figure>

The significative others set ($O_s$) are the users that receive a significant larger amount of retweets than the rest of others. Could be that the significative others set is empty, and thus we don't have an ego network for that particular user.

For the definition of $\text{Outliers}$ we will use what is stated in the paper [[3]](#ref3), which introduces the skewness into the measurement of the outliers. With 
$$
Q_i := i\text{th quartile P} \\
IQR:=Q_3-Q_1\\
MC := \text{Medcouple(P)}
$$
We define:

$$
\text{Outliers}(P)=\{x|x\notin[Q_1 − h_l(MC) IQR; Q_3 + h_u(MC) IQR]\}\\
h_l(MC) = 1.5e^{-4MC}\\
h_u(MC) = 1.5e^{3MC}
$$
In our case we will only use the upper bound ($\text{OutlierFrontier}(P)=Q_3 + h_u(MC) IQR$), as we have a long tail distribution, with the most frequently retweeted profiles in the right. 

The *ego-significant-network* is the directed network with with the following nodes: the ego ($u$), and the outliers of its retweet distribution ($O_s(u)$). We will have a directed edge between $u$ and $v$ if $v\in O_s(u)$. Of course the ego ($u$) will have outgoing edges to all the other nodes in the graph. For every outlier $o_i\in O_s(u)$, we find its outliers ($O_s(o_i)$), and if the $o_{o_i}\in O_s(u)\cup{u}$ we will add an edge from $o_i$ to $o_{o_i}=o_j\in O_s(u)\cup{u}$.

<figure style="text-align:center">
    <iframe frameborder="0" style="width:100%;height:400px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Copy%20of%20EgoNetwork.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1sr0CH8XmM9-gACD6xLYHk06G6z61Bpmt%26export%3Ddownload"></iframe>
<figcaption>Fig.2 - Example of ego-network.</figcaption>
</figure>

### Example

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='https://bernatesquirol.github.io/tfm-plots/ego-networks-slider' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.3 - Fitting a gamma distribution to degree of the ego-networks for different users.</figcaption>
</figure>



## Timeline networks analysis

We will construct a significant ego network for each user timeline, and we will look at overall properties of these ego-networks. As we don't have a balanced number of users for each type, we will apply a bootstrapping technique in order to compare the distributions. This means that we will pick a random subsample of the distribution of the same size for each user group, compare among them and compute the mean of the comparisons. The plot of the histogram of the variables associated with the networks will be done in a similar fashion.

### Degree

The first variable we will look at is the degree of the ego inside the network, that is, the total number of nodes of the network minus 1 (the ego). We can see how the politicians differ very much from the other groups in $\text{Fig.4}$.
$$
\text{Degree}(G_u)={\#G_u-1}
$$

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/networks-degree-type.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.4 - Degree of the ego-network by type.</figcaption>
</figure>



**P-values for sampled Kolmogorov-Smirnov test **

|                     | journalist | politician | random-follower | random-friend |
| ------------------: | ---------: | ---------: | --------------: | ------------: |
|      **journalist** |   0.013974 |   0.275131 |        0.117046 |      0.060879 |
|      **politician** |   0.274419 |   0.041206 |        0.184481 |      0.257700 |
| **random-follower** |   0.118161 |   0.185019 |        0.008606 |      0.078555 |
|   **random-friend** |   0.060738 |   0.259250 |        0.078369 |      0.008491 |

If we fit a $\text{Gamma}$ distribution to the histograms of the different user types, we find the results plotted in $\text{Fig.4}$ and $\text{Fig.5}$.

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='https://bernatesquirol.github.io/tfm-plots/degrees-slider' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'></iframe>
    <figcaption>Fig.4 - Fitting a gamma distribution to degree of the ego-networks for different users.</figcaption>
</figure>

<figure style="text-align:center">
    <img src='https://bernatesquirol.github.io/tfm-plots/static/degree-fits.png' height=250>
    <figcaption>Fig.5 - Gamma fits plotted</figcaption>
</figure>


We can see how the plot of the fitted distributions looks like in $\text{Fig.5}$. This means politicians tend to have more outliers than other user types. We can also see that the QQ-plots in journalists and random users are very similar, the $\text{Gamma}$ distribution fails in a similar way.

### Reciprocity

We define reciprocity as the number of outliers that have edges towards the ego divided by the amount of outliers.
$$
\text{Reciprocity}(G_u)=\frac{\#\{v\in{(G_u-u)}|(v,u)\in \text{Edges}(G_u)\}}{\#G_u-1}
$$

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/networks-ego-reciprocity-type.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.6 - Ego reciprocity by type.</figcaption>
</figure>

**P-values for sampled Kolmogorov-Smirnov test**

|                 | journalist | politician | random-follower | random-friend |
| --------------: | ---------: | ---------: | --------------: | ------------: |
|      journalist |   0.010489 |   0.105550 |        0.040881 |      0.105750 |
|      politician |   0.103113 |   0.034175 |        0.132675 |      0.157906 |
| random-follower |   0.040887 |   0.130812 |        0.006023 |      0.125938 |
|   random-friend |   0.105946 |   0.157087 |        0.125920 |      0.007257 |

We see no clear differences between reciprocity in each type of user, as nearly $80$% of the users don't have reciprocity at all from their outliers. That proportion is a bit lower with politicians.

### Connections between outliers

We define outlier connectedness as the number of edges between outliers, divided by the number of possible edges between outliers.
$$
\text{OutlierConnectedness}(G_u)=\frac{\#\{v\in (G_u-{u})|\exist w\in (G_u-{u}),(v,w)\in \text{Edges}(G_u)\}}{(\#G_u-1)\times(\#G_u-2)}
$$

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/networks-outlier-connectedness.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.7 - Connections between outliers.</figcaption>
</figure>

**P-values for sampled Kolmogorov-Smirnov test **

|                     | journalist | politician | random-follower | random-friend |
| ------------------: | ---------: | ---------: | --------------: | ------------: |
|      **journalist** |   0.016117 |   0.469709 |        0.121097 |      0.059129 |
|      **politician** |   0.469169 |   0.048047 |        0.558213 |      0.488959 |
| **random-follower** |   0.120751 |   0.559362 |        0.009477 |      0.076149 |
|   **random-friend** |   0.059404 |   0.488519 |        0.076078 |      0.009497 |

Connections between outliers are way more common among politicians, as their outliers are probably other politicians in the same party.

### Ego clustering

We will use the following definition of clustering for directed networks, found in [**[5]**](#ref5):
$$
\text{EgoClustering}(G_u)=c_{u}=\frac{1}{\operatorname{deg}^{\operatorname{tot}}(u)\left(\operatorname{deg}^{\operatorname{tot}}(u)-1\right)-2 \operatorname{deg}^{\leftrightarrow}(u)} T(u)
$$
where $T(u)$ is the number of directed triangles through node $u$, $\operatorname{deg}^{\operatorname{tot}}(u)$ is the sum of in degree and out degree of $u$ and $\operatorname{deg}^{\leftrightarrow}(u)$ is the reciprocal degree of $u$, and $u$ is the ego.

<figure style="text-align:center">
    <iframe height='520' scrolling='no' src='../tfm-plots/networks-ego-clustering-type.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.8 - Clustering score for ego-node.</figcaption>
</figure>


**P-values for sampled Kolmogorov-Smirnov test **

|                     | journalist | politician | random-follower | random-friend |
| ------------------: | ---------: | ---------: | --------------: | ------------: |
|      **journalist** |   0.014539 |   0.580375 |        0.110850 |      0.087493 |
|      **politician** |   0.579250 |   0.049219 |        0.664144 |      0.630919 |
| **random-follower** |   0.111391 |   0.664238 |        0.008437 |      0.036767 |
|   **random-friend** |   0.085648 |   0.631700 |        0.036911 |      0.008309 |

In fact ego clustering and outlier connectedness are correlated because the ego has forward edges to all other nodes. The more connections between outliers, the more triangles there will be through ego (more clustering).

## Evolving network

In this section we will measure changes in the ego-network as activity changes. We will split user's timeline into the temporal chunks ($\tau_i$) we found in the [linear breakpoints section]() in order to have some stationarity in the data. We will then compute where  the outlier frontier is for this temporal interval $\tau_i$. We will then divide it by the total retweeted amount  (${\sum_{v\in P_{\tau_i}}{R_u(v)}}$) during $\tau_i$, having as a result the proportion of retweets ($\sigma_{\tau_i}$) needed from a user to become outlier during $\tau_i$. Then we will input this proportion ($\sigma_{\tau_i}$) in the weekly distribution of retweet counts ($P_j$), week by week during the breakpoints, which will result in user's weekly outliers.
$$
\begin{align*}
\text{for }  i &= 1\ldots \#breakpoints: \\

\sigma_{\tau_i}&=\frac{\text{OutlierFrontier}(P_{\tau_i})}{\sum_{v\in P_{\tau_i}}{R_u(v)}}\\
 T_i &\sim \text{Poisson}(\text{rate}=\lambda_i)\\
 &\text{for }  j = 1\ldots \#weeks: \\
 & \text{Outliers}'_{ij}(P_j)=\left\{o|\frac{R_u(o)}{\sum_{v\in P_j} R_u(v)}>\sigma_{\tau_i}\right\}
 \end{align*}
$$

Within this framework, in order to measure the stability of the ego-network, we will compute the following features for every week:

- Number of outliers: number of outliers that fall in the higher end of the frontier in the retweet counts per week.

- Number of new outliers: the number of new outliers that we have not seen before in previous weeks. This number will be higher at the beginning of the study, as we won't have seen that much profiles.

- Jaccard index: the Jaccard index is computed with the formula:
  $$
  J(\text{Outliers}'_{i,j},\text{Outliers}'_{i,j+1})=\frac{|\text{Outliers}'_{i,j}\cap\text{Outliers}'_{i,j+1}|}{|{Outliers}'_{i,j}\cup\text{Outliers}'_{i,j+1}|}
  $$

### Example

We will analyse the evolution of congresswoman Inés Arrimadas (Ciudadanos) ego network. In the period of time`15/08/2019-11/11/2019` we encounter, nearly every week, Albert Rivera, the leader of the party, in its outliers. Then `11/11/2019` Albert Rivera, announces its retirement from politics and steps down as the leader party, other outliers become more important, and we can see how the Jaccard index decreases and the new outliers per week increases in the following months. But we also observe that in confinement period the behaviour of those variables changes a lot more than with the leader resignation.

Example of the network evolution of Inés Arrimadas:

|           Week | jaccard  | New outliers | Total outliers | Outliers                                            |
| -------------: | -------- | ------------ | -------------- | --------------------------------------------------- |
|     2019-09-30 | 0.5      | 0            | 3              | [Albert_Rivera, CiudadanosCs, Lroldansu]            |
|     2019-10-07 | 0.666667 | 0            | 2              | [CiudadanosCs, Albert_Rivera]                       |
|            ... |          |              |                |                                                     |
| **2019-11-11** | 0        | 0            | 0              | []                                                  |
|     2019-11-18 | 0        | 1            | 3              | [CiudadanosCs, **davidmartinezg**, carrizosacarlos] |
|     2019-11-25 | 0        | 1            | 1              | [**JuanMarin_Cs**]                                  |

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/networks-ines-1.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.9 - Example of jaccard index for a timeline of the congresswoman Inés Arrimadas</figcaption>
</figure>
<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/networks-ines-2.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.10 - Number of weekly outliers by Inés Arrimadas.</figcaption>
</figure>

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/networks-ines-3.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.11 - RT and text count for Inés Arrimadas for every week.</figcaption>
</figure>



### Global analysis

We would like to see if our example's observed behaviour can also be found in a global scale for all the users we have in the dataset. If we look at the overall picture of the Jaccard index, we observe that, although it is very smooth as we have a lot of users, there are lower values in the end of December and in the beginning of March. If we observe those dates in $\text{Fig.13}$, we see two different behaviours. While in December users have less outliers per week, probably because they retweeted less, in March there is a peak in the number of outliers of our users.

<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/networks-jaccard-corrected-total.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.12 - Clustering score for ego-node.</figcaption>
</figure>



<figure style="text-align:center">
    <iframe height='320' scrolling='no' src='../tfm-plots/networks-count-outlier-corrected.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.13 - Clustering score for ego-node.</figcaption>
</figure>


If we only look at politicians and journalist, this phenomenon is amplified, probably because we have less profiles and less variance.

<figure style="text-align:center">
    <iframe height='435' scrolling='no' src='../tfm-plots/networks-jaccard-corrected-types.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.14 - Clustering score for ego-node.</figcaption>
</figure>



<figure style="text-align:center">
    <iframe height='820' scrolling='no' src='../tfm-plots/networks-count-outlier-corrected-types.html' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 99.99%;'>	</iframe>
    <figcaption>Fig.15 - Clustering score for ego-node.</figcaption>
</figure>




## References

<a name="ref1">**[ 1 ]**</a>: Constantine Sedikides, Marilynn B. Brewer. *Individual Self, Relational Self, Collective Self*. https://doi.org/10.4324/9781315783024 (https://www.taylorfrancis.com/books/9781315783024)

<a name="ref2">**[ 2 ]**</a>: Valerio Arnaboldi, Andrea Passarella, Marco Conti, Robin Dunbar. *Structure of Ego-Alter Relationships of Politicians in Twitter*. https://doi.org/10.1111/jcc4.12193 (https://academic.oup.com/jcmc/article/22/5/231/4666424)

<a name="ref3">**[ 3 ]**</a>: M. Hubert, E. Vandervieren. *An adjusted boxplot for skewed distributions*, Computational Statistics & Data Analysis, Volume 52, Issue 12, 2008, Pages 5186-5201, ISSN 0167-9473. https://doi.org/10.1016/j.csda.2007.11.008. (http://www.sciencedirect.com/science/article/pii/S0167947307004434)

<a name="ref4">**[ 4 ]**</a>: Valerio Arnaboldi, Marco Conti, Andrea Passarella, Robin Dunbar. *Online Social Networks and information diffusion: The role of ego networks*, Online Social Networks and Media, Volume 1, 2017, Pages 44-55, ISSN 2468-6964, https://doi.org/10.1016/j.osnem.2017.04.001 (http://www.sciencedirect.com/science/article/pii/S2468696417300150)

<a name="ref5">**[ 5 ]**</a>: Giorgio Fagiolo. *Clustering in complex directed networks*. https://doi.org/10.1103/PhysRevE.76.026107 (https://journals.aps.org/pre/abstract/10.1103/PhysRevE.76.026107)