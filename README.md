# CS4220 Machine Learning 1 Lab Exercises

### *Delft University of Technology 23/24*


## 1. Basics of Machine Learning

- **Optimal decision boundary** (where posteriors $P(y1|x) = P(y2|x)$) corresponds to **Bayes error** (i.e. minimal error)
-  **Quadratic Classifier (QDA)** has a quadratic decision boundary, due to limited data. 
    - In the limit, the solutions by LDA and QDA will coincide
- Hyperparameters (such as $h$ in Parzen estimator) should be tuned on test sets independent from training data
    - Optimal value can be found by plotting values (x-axis) against the log-likelihood (y-axis) using elbow method
- k-NN and NMC are sensitive to **scaling** of features, because these methods do not estimate covariance matrices


## 2. Linear Regression & Linear Classifiers

- Differently from labelled datasets (often numeric labels $y$) in supervised classification tasks, **regression datasets** contain inputs $x$ (uniformly distributed e.g.) with a corresponding $y$ depdendent on $x$ ($y = x^2 + \epsilon$ where $\epsilon$ can be some Gaussian noise e.g.)
- We can fit linear functions under the **squared loss** using $w = (X^TX)^-1 * X^T * Y$
    - Typically we have an intercept as well giving $w^T = [w_0, w_1]$ which can be plugged into $f(x) = w_1 * x + w_0$ 
    - Note we need to add a bias term to X
- **Polynomial Regression** fits a polynomial of some *maximal* degree to the data in a least squares sense
    - *Note: even though this results in a non-linear function in x, the regression function is linear in the unknown parameters w estimated from the data.*
    - If there are 4 data points, one needs at least a third-order polynomial to fit these (whether there is a bias term does not matter)
    - Note that linear regressors (`linearr`) do not take into cross-terms, leading to high error rates when trained on (non-linear data) that contain multiplications of $x_i$ terms: e.g. data modelled as $y = sin(x1)sin(x2)$
- **Decision boundary** of **linear classifier** is found at $f(x) = y = 0$
    - $y = ax + b$ with intercept $b = -w_0 / w_2$ and slope $a = -(w_0/w_2)/(w_0/w_1)$


## 3. Losses, Regularization, Evaluation

- Regularization aims to stabilize objective functions to prevent overfitting - possibly caused by curse of dimensionality or lack of data (multicollinearity)
    - **Ridge regularization (L2)** keeps weights small
    - **Lasso regularization (L1)** introduces sparsity and can get rid of redundant features
    - Whereas $w_i$ entries can become 0 for large $\lambda$ (i.e. small $\tau$) in L1 regularization, this is impossible for L2
- The optimal regularization parameter $\lambda$ can be found using **cross-validation** (hyperparameter tuning)
    - **K-fold Cross Validation**: N chunks used for training independent classifiers (take avg of $N$ error estimates), 1 chunk used for evaluating (compared against *true error* on full training set)
    - **Leave-one-out-Procedure**: Single test object
- **Learning curves** show classification errors against the training set size
    - The discrepancy between *true error* and *apparent error* is **overfitting**
- **Feature curves** show how the classification error varies with varying numbers of feature dimensionality
    - **Curse of dimensionality** = error goes up after a certain dimensionality threshold 
- *Variabililty in error* is larger for small training sets
    - The more complex the model, the more training samples needed! 
    - *Large, independent test sets* yield an unbiased and small variance error estimate (but worse classifier due to less data used for training..)
    - *Small, independent test sets* yield an unbiased and large variance error estimate



## 4a. Probabilistic Models 

- **Maximum Likelihood** finds the parameters $w$ that maximize $P(x, y | w)$, i.e. probability of observed data
- **Maximum A Posteriori** finds $w$ that maximize $P(w | x, y) = P(x, y | w) * P(w)$, i.e. it assumes some prior knowledge about the weights in $P(w)$ - e.g. uniform in $[a,b]$ or Gaussian distribution $N(w | \mu, \sigma^2)$
    - Prior increases bias, but reduces variance
    - The *Haldane prior* is an improper prior that does not satisfy all properties of a pdf
        - Leads to uniform posterior $P(w | x, y)$, so no unique MAP solution
    - $w_{MAP}$ may coincide with the solutions for Ridge (R2) and Lasso (L1) regularization, depending on assumptions made on the prior

- **Bayesian networks** allow us to reason about dependencies between random variables and thus construct $P$ from simpler components 
    - Given $N$ RVs, there are $N!$ possible decompositions
    - Even if not all assumptions are valid, model complexity is still lowered due to fewer parameters to estimate 


## 4b. Clustering

- Clustering aims to find patterns in unstructured data
- **Agglomerative Hierarchical Clustering**
    - Input: dataset `X`, distance matrix `D` and linkage type (single, average complete)
    - Output: dendogram
    - Algorithm stops when there is 1 cluster left, after which we cut the dendogram to obtain the desired no. of clusters (long vertical bars imply large distances)
    - *Single linkage is sensitive to outliers!*
- **Mixture of Gaussians**
    -  We assume K separate distributions, one for each cluster
    - The  **EM-algorithm** is used to approximate model parameters
        - E-step: Update membership $P(C_k|x; \theta)$ based on updated classifier
        - M-step: Improve model by updating maximum likelihood estimates ($L(\theta | x) = \prod^N_{i=1}p(x_i|\theta)$) of parameters based on cluster membership
- **Cluster validation** is used for assessing clusterings and no. of clusters
    - **Fusion level maps** plot the linkage distances (y-axis) against the number of clusters (x-axis)
        - Heuristic: cut the dendogram at largest jump
    - **Davies-Bouldin Index (DBI)** is a cluster score that incorporates both **within-** and **between scatters**
        - Heuristic: pick the no. of clusters that minimizes the DBI (minimum in plot)

### Partitional Techniques

#### K-means

| Pros | Cons |
| -------- | -------- |
| Simple, fast     | <span style="color:red"> Assumes spherical/convex clusters </span> | 
| | <span style="color:red"> Sensitive to initialization <span> | 
|  | Can get stuck in local minima (*start from many random initializations*) |
| | Clusters can lose all samples (*remove cluster or split largest into 2*) | 


#### Mixture of Gaussians (soft / probabilistic)

| Pros | Cons |
| -------- | -------- |
| Can use prior knowledge on cluster distribution | <span style="color:red"> Assumes a priori known no. of clusters </span> |
| Gives general framework for any density mixture | Need to define a cluster density (e.g. Gaussian) |
| Allows for overlapping clusters | Guarantees finding of *local* optimum only |
| | May converge slowly |
| | <span style="color:red"> Is dependent on initialization <span> | 



#### Mean Shift

| Pros | Cons |
| -------- | -------- |
| Does not assume spherical/convex clusters | Computationally expensive |
| Just a single parameter (window size) | Output depends on window size |
| Finds variable no. of modes | Does not scale well with dimension of feature space |
| Robust to outliers | |2+++++++++++++++++++++++++++++++++++++++++++++++++ 
| Finds global optimum | |



### Hierarchical Techniques

| Pros | Cons |
| -------- | -------- |
| Dendogram gives intuitive overview of possible clusters | Single linkage is sensitive to outliers |
| Linkage type allows for clusters of varying shapes (convex and non-convex) | Computationally expensive! $O(n^2)$ in time and space |
| Different dissimilarity measures can be used | Clusterings limited to "hierarchical nestings" |



### Graph Techniques

#### Spectral Clustering

| Pros | Cons |
| -------- | -------- |
| Can operate on arbitrary graphs | <span style="color:red"> Assumes a priori known no. of clusters </span>  |
| Clusters can have any shape | Requires a K-means clustering anyways |
| All no. of clusters are possible | Computationally relatively expensive ($O(N^3)$ for non-sparse matrices) |





## 5. Complexity

- The **Support Vector Machine (SVM)** is a linear classifier that aims to minimize the VC-dimension (complexity metric) by constrained optimization
    - $g(x) = w^Tx + w_0 = \sum^{N}_{i=1}{\alpha_iy_ix_i} + w_o$ 
- Its solution $w = \sum^{N}_{i}{a_i}y_ix_i$ is expressed in terms of objects ("constraints"), rather than features
    - We refer to those objects with $a_i > 0$ as support vectors - only these have influence on the classifier
- *Problem 1: data should be linearly separable*
    - Idea: introduce **slack variables** $\varepsilon$ to weaken the constraints of the optimization problem
- *Problem 2: decision boundary should be linear*
    - Idea: use the **kernel trick** to map data to high dimensional feature space - allows for varying complexity 
    - $K(x, y) = \Phi(x)^T\Phi(y)$ 
    - Polynomial kernel: $K(x,y) = (x^Ty + 1)^d$
    - Gaussian / Radial Basis (RDF) kernel: $K(x,y) = e^{-\frac{|||x-y||^2}{\sigma^2}}$
        - Basically a weighted version of Parzen density estimator! However, faster since we only care about the support vectors (not all data points) 



## 6a. Feature Reduction

### Feature Extraction

**Principle Component Analysis** - Unsupervised

- Does not directly reduce dimensions, but projects data to new space i.e. moves x-axis into principle axis of largest variation (`PC 1`) by translation and rotation

- Global and linear

- Can only be applied to data that is linearly separable
    - If that is not the case, a kernel function $\Phi$ can be used to map original data to higher dimension in which it's linearly separable 


**Linear Discriminant Analysis (Fisher Mapping)** - Supervised

- Similarity with PCA: projects data onto new axes to reduce dimensionality
- Difference: maximizes *separability of classes* rather than *global variance*



### Feature Selection

Greedy approximations (alternative to exhaustive search algorithms):

- **Simplest Variation**: select best individual $d$

- **Forward Selection**: start with empty feature set. One at a time, keep adding feature that gives best performance when added to current selection.

- **Backward Selection**: start with all features. One at a time, remove feature that leads to best performance considering the feature selection remaining.

- **Plus-l-take-away-r**: start with empty set (if $l > r$) or entire set (if $l < r$). Keep adding best $l$ and removing worst $r$ or vice versa.

Criteria to use in approximations above:

- **Heuristic Scatter Based**

$J_1 = trace(S_w + S_B) = trace(\sum)$

$J_2 = ...$ 

- **Mahalanobis Distance**: distance measure between point and *distribution* - unlike Euclidean, accounts for variability in features!

$D_M = (\mu_1 - \mu_2)^TC^-1(\mu_1 - \mu_2)$


- **Fisher Criterion**: seeks to minimize within-scatter and maximize between-scatter

$J_F = \frac{(\mu_1 - \mu_2)^2}{\sigma_1^2 + \sigma_2^2}$



## 6b. Combining Classifiers

### Fixed Combining Rules

- Special trained rules based on "classifier confidences"
- General trained rules interpreting base-classifier outputs as features

Base classifier: $y = S(x | \theta_{base})$

Combining classifier: $z = C(y | \theta_{comb})$

Different combining rules possible: 
- Product, minimum
- Sum (mean), median, majority vote
- Maximum

<span style="color:red"> Supoptimal: neglect the classification confidence characteristic of the base classifier outputs, as they are treated as general feature values... </span>



### Trained Combining Rules

Output: posterior probability distribution over all $C$ classes, rather than a single class decided by a fixed combining rule.

Special trained combiners:
- DT: Decision Templates ~ Nearest Mean
- BKS: Behavioral Knowledge Space
- DCS: Dynamic Classifier Selection
- ECOC: Error Correcting Output Coding
- NN: Neural Networks


#### Generation of Base Classifiers

- **Random Subspace** Approach
- **Bagging** ~ bootstrapping & aggregate
- **Boosting ~ AdaBoost**
    - Assumes decision stump as weak base classifier
    - Minimizes exponential error
    - <span style="color:red"> Suffers from overtraining when combining ~ >= 1000 classifiers... </span> - large discrepancy between true error and apparent error (never converge)