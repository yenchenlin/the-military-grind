---
layout: post
title: 圖書館 or 健身房?
---
來到兵器連認識了 Zack, Vibert 等人以後, 寢室就變成了他們的健身房. 而我與晨勃, 屈暄等人
則把寢室當成貨真價實的圖書館.

## Markov Random Fields

{% marginfigure 'nb1' 'assets/img/mrf.png' 'Undirected graphical representation of a joint probability of voting preferences over four individuals. The figure on the right illustrates the pairwise factors present in the model.'%} As a motivating example, suppose that we are modeling voting preferences among persons $$A,B,C,D$$. Let's say that $$(A,B)$$, $$(B,C)$$, $$(C,D)$$, and $$(D,A)$$ are friends, and friends tend to have similar voting preferences. These influences can be naturally represented by an undirected graph.

One way to define a probability over the joint voting decision of $$A,B,C,D$$ is to assign scores to each assignment to these variables and then define a probability as a normalized score. A score can be any function, but in our case, we will define it to be of the form
{% math %}
\tilde p(A,B,C,D) = \phi(A,B)\phi(B,C)\phi(C,D)\phi(D,A),
{% endmath %}
where $$\phi(X,Y)$$ is a factor that assigns more weight to consistent votes among friends $$X,Y$$, e.g.:
{% math %}
\begin{align*}
\phi(X,Y) =
\begin{cases}
10 & \text{ if}\; X = Y = 1 \\
5 & \text{ if}\; X = Y = 0 \\
1 & \text{ otherwise}.
\end{cases}
\end{align*}
{% endmath %}
The factors in the unnormalized distribution are often referred to as *factors*.
The final probability is then defined as
{% math %}
p(A,B,C,D) = \frac{1}{Z} \tilde p(A,B,C,D),
{% endmath %}
where $$ Z = \sum_{A,B,C,D} \tilde p(A,B,C,D) $$ is a normalizing constant that ensures that the distribution sums to one.

When normalized, we can view $$\phi(A,B)$$ as an interaction that pushes $$B$$'s vote closer to that of $$A$$. The term $$\phi(B,C)$$ pushes $$B$$'s vote closer to $$C$$, and the most likely vote will require reconciling these conflicting influences.

Note that unlike in the directed case, we are not saying anything about how one variable is generated from another set of variables (as a conditional probability distribution would do). We simply indicate a level of coupling between dependent variables in the graph. In a sense, this requires less prior knowledge, as we no longer have to specify a full generative story of how the vote of $$B$$ is constructed from the vote of $$A$$ (which we would need to do if we had a $$P(B\mid A)$$ factor). Instead, we simply identify dependent variables and define the strength of their interactions; this in turn defines an energy landscape over the space of possible assignments and we convert this energy to a probability via the normalization constant.

### Formal definition

A Markov Random Field (MRF) is a probability distribution $$p$$ over variables $$x_1,...,x_n$$ defined by an *undirected* graph $$G$$ in which nodes correspond to variables $$x_i$$. The probability $$p$$
has the form
{% math %}
p(x_1,..,x_n) = \frac{1}{Z} \prod_{c \in C} \phi_c(x_c),
{% endmath %}
where $$C$$ denotes the set of *cliques* (i.e. fully connected subgraphs) of $$G$$.
The value
{% math %}
Z = \sum_{x_1,..,x_n}\prod_{c \in C} \phi_c(x_c)
{% endmath %}
is a normalizing constant that ensures that the distribution sums to one.

Thus, given a graph $$G$$, our probability distribution may contain factors whose scope is any clique in $$G$$, which can be a single node, an edge, a triangle, etc. Note that we do not need to specify a factor for each clique. In our above example, we defined a factor over each edge (which is a clique of two nodes). However, we chose not to specify any unary factors i.e. cliques over single nodes.

### Comparison to Bayesian networks

{% marginfigure 'nb1' 'assets/img/mrf2.png' 'Examples of directed models for our four-variable voting example. None of them can accurately express our prior knowledge about the dependency structure among the variables.'%}
In our earlier voting example, we had a distribution over $$A,B,C,D$$ that satisfied $$A \perp C \mid  \{B,D\}$$ and $$B \perp D \mid  \{A,C\}$$ (because only friends directly influence a person's vote). We can easily check by counter-example that these independencies cannot be perfectly represented by a Bayesian network.
However, the MRF turns out to be a perfect map for this distribution.

More generally, MRFs have several advantages over directed models:

- They can be applied to a wider range of problems in which there is no natural directionality associated with variable dependencies.
- Undirected graphs can succinctly express certain dependencies that Bayesian nets cannot easily describe (although the converse is also true)

They also possess several important drawbacks:

- Computing the normalization constant $$Z$$ requires summing over a potentially exponential number of assignments. We will see that in the general case, this will be NP-hard; thus many undirected models will be intractable and will require approximation techniques.
- Undirected models may be difficult to interpret.
- It is much easier to generate data from a Bayesian network, which is important in some applications.

It is not hard to see that Bayesian networks are a special case of MRFs with a very specific type of clique factor (one that corresponds to a conditional probability distribution and implies a directed acyclic structure in the graph), and a normalizing constant of one. In particular, if we take a directed graph $$G$$ and add side edges to all parents of a given node (and removing their directionality), then the CPDs (seen as factors over a variable and its ancestors) factorize over the resulting undirected graph. The resulting process is called *moralization*.
{% maincolumn 'assets/img/moralization.png' 'A Bayesian network can always be converted into an undirected network with normalization constant one. The converse is also possible, but may be computationally intractable, and may produce a very large (e.g. fully connected) directed graph.' %}

Thus, MRFs have more power than Bayesian networks, but are more difficult to deal with computationally. A general rule of thumb is to use Bayesian networks whenever possible, and only switch to MRFs if there is no natural way to model the problem with a directed graph (like in our voting example).

## Independencies in Markov Random Fields

Recall that in the case of Bayesian networks, we defined a set of independencies $$I(G)$$ that were described by a directed graph $$G$$, and showed how these describe true independencies that must hold in a distribution $$p$$ that factorizes over the directed graph, i.e. $$I(G) \subseteq I(p)$$.

{% marginfigure 'markovblanket' 'assets/img/markovblanket.png' 'In an MRF, a node $$X$$ is independent from the rest of the graph given its neighbors (which are reffered to at the Markov blanket of $$X$$.'%}
What independencies can be then described by an undirected MRF? The answer here is very simple and intuitive: variables $$x,y$$ are dependent if they are connected by a path of unobserved variables. However, if $$x$$'s neighbors are all observed, then $$x$$ is independent of all the other variables, since they influence $$x$$ only via its neighbors.

In particular, if a set of observed variables forms a cut-set between two halves of the graph, then variables in one half are independent from ones in the other.

{% maincolumn 'assets/img/cutset.png' '' %}

Formally, we define the *Markov blanket* $$U$$ of a variable $$X$$ as the minimal set of nodes such that $$X$$ is independent from the rest of the graph if $$U$$ is observed, i.e. $$X \perp (\mathcal{X} - \{X\} - U) \mid  U$$. This notion holds for both directed and undirected models, but in the undirected case the Markov blanket turns out to simply equal a node's neighborhood.

In the directed case, we found that $$I(G) \subseteq I(p)$$, but there were distributions $$p$$ whose independencies could not be described by $$G$$. In the undirected case, the same holds. For example, consider a probability described by a directed v-structure (i.e. the explaining away phenomenon). The undirected model cannot describe the independence assumption $$X \perp Y$$.

{% maincolumn 'assets/img/mrf-bn-comparison.png' 'Examples of probability distributions that have a perfect directed graphical representation but no undirected representation, and vice-versa.' %}


## Conditional Random Fields

An important special case of Markov Random Fields arises when they are applied to model a conditional probability distribution $$p(y\mid x)$$. In this case, $$x \in \mathcal{X}$$ and $$y \in \mathcal{Y}$$ are vector-valued variables; we are typically given $$x$$ and want to say something interesting for $$y$$. Typically, distributions of this sort will arise in a supervised learning setting, where $$y$$ will be a vector-valued label that we will be trying to predict. This setting is typically referred to as *structured prediction*.

### Example



## 不運動, 我會死

It is often useful to view MRFs in a way where factors and variables are explicit and separate in the representation. A factor graph is one such way to do this. A factor graph is a bipartite graph where one group is the variables in the distribution being modeled, and the other group is the factors defined on these variables. Edges go between factors and variables that those factors depend on.

{% maincolumn 'assets/img/factor-graph.png' 'Example of a factor graph with three variables and four factors.' %}

This view allows us to more readily see the factor dependencies between variables, and later we'll see it allows us to compute some probability distributions more easily.

<br/>

|[Index](../../) | [Previous](../directed) |  [Next](../../inference/ve)|
