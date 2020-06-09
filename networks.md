## Ego network analysis

In this section will discuss a way of creating *ego-networks* based on

### Concepts

Given a user $u$, let's define:

- $O(u)$: Others of $u$. The set of users that recibe 2 or more retweets from $u$.
- $R_u(o)$: Number of retweets that recieves a users $o\in O(u)$ from $u$
- $P(R_u)=P(\{ R_u(o), \forall o \in O(u)\})$: Empíric distribution of $R_u(o)$ for all $o\in O(u)$.
- $O_s(u)=\{o\in O(u)| R_u(o)\in \text{Outliers}(P)\}\sub O(u)$: Significative others of $u$. The subset of users for which its number of retweets from $u$ represents an outlier of the $P(R_u)$ distribution

<figure style="text-align:center">
    <iframe frameborder="0" style="width:100%;height:400px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=EgoNetwork.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D13_1OP3DQnsCCYWEf-_Nvcd80MBNHQ6kb%26export%3Ddownload"></iframe>
<figcaption>Fig.1 - Schema of ego-network.</figcaption>
</figure>

The significative others set ($O_s$) are the users that recieve a significant larger amount of retweets than the others. 

### Evolving network

- stability through time (pre covid)
- 

### Users

