Gaussian Mixture Models
-----------------------

A multivariate normal distribution or multivariate Gaussian distribution is a generalization of the one-dimensional
Gaussian distribution into muliple dimensions.

.. math::
     \mathcal{N}(\mathbf{x}|\mathbf{\boldsymbol\mu}, \mathbf{\Sigma}) =
    \frac{1}{(2 \pi)^{D/2}}
    \frac{1}{ | \mathbf{\Sigma} |^{1/2} }
    exp \{ -\frac{1}{2} (\mathbf{\mathbf{x}}-\mathbf{\boldsymbol\mu)}^T \mathbf{\Sigma}^{-1} (\mathbf{\mathbf{x}}-\mathbf{\boldsymbol\mu)} \}

The distribution is given by its mean, :math:`\mathbf{\boldsymbol\mu}`, and covariance, :math:`\mathbf{\Sigma}`, matrices.
To generate samples from the multivariate normal distribution under python, one could use the 
`numpy.random.multivariate_normal <http://docs.scipy.org/doc/numpy/reference/generated/numpy.random.multivariate_normal.html>`_ function from numpy.

In statistics, a mixture model is a probabilistic model for density estimation using a mixture distribution. A mixture model can be regarded as a type of unsupervised learning or clustering [wikimixmodel]_.
Mixture models provide a method of describing more complex propability distributions, by combining several probability distributions.
Mixture models can also be used to cluster data.

The Gaussian mixture distribution is given by the following equation [bishop2006]_:

.. math::
    p(\mathbf{x}) = \sum_{k=1}^{K} \pi_k \mathcal{N} (\mathbf{x} | \boldsymbol\mu_k, \mathbf{\Sigma}_k)

Here we have a linear mixture of Gaussian density functions, :math:`\mathcal{N} (\mathbf{x} | \boldsymbol\mu_k, \mathbf{\Sigma}_k)`.
The parameters :math:`\pi_k` are called mixing coefficients, which must fulfill

.. math::
    \sum_{k=1}^K \pi_k = 1

and given :math:`\mathcal{N} (\mathbf{x} | \boldsymbol\mu_k, \mathbf{\Sigma}_k) \geq 0$ and $p(\mathbf{x}) \geq 0` we also have that

.. math::
    0 \geq \pi_k \geq 1

Sampling and Probability Density Function
^^^^^^^^

PyPR has some simple support for sampling from Gaussian Mixture Models.

.. automodule:: pypr.clustering.gmm
   :members: sample_gaussian_mixture

The example *sample_gmm* gives an example of drawing samples from a mixture of three clusters, and plotting the result.

.. literalinclude:: ../../examples/sample_gmm.py

The resulting plot should look something like this:

.. image:: figures/sample_gmm.*

..
    Other probability distributions can be used instead of the Gaussian, but PyPR does not support this.

The *probability denisity function* (PDF) can be evaluated using the following function:

.. automodule:: pypr.clustering.gmm
   :members: gmm_pdf

For more information mixture models and expectation maximization take a look at Pattern Recognition and Machine Learning by Christopher M. Bishop [bishop2006]_.

.. [wikimixmodel] Mixture model. (2010, December 23). In Wikipedia, The Free Encyclopedia. Retrieved 12:33, February 25, 2011, from http://en.wikipedia.org/w/index.php?title=Mixture_model&oldid=403871987

.. [bishop2006] Pattern Recognition and Machine Learning, Christopher M. Bishop, Springer (2006)

Expectation Maximization
^^^^^^^^^^^^^^^^^^^^^^^^

In the previous example we saw how we could draw samples from a Gaussian Mixture Model.
Now we will look at how we can work in the opposite direction, given a set of samples find a set of K multivariate Gaussian
distributions that represent observed samples in a good way.
The number of clusters, :math:`K`, is given, so the parameters that are to be found are the means and covariances of the distributions.
The Expectation Maximization (EM) algorithm will be used to find these parameters 

The EM algorithm is an iterative refinement algorithm used for finding maximum likelihood estimates of parameters in probabilistic models.
The likelihood is a measure for how good the data fits a given model, and is a function of the parameters of the statistical model.
If we assume that all the samples in the dataset are independent, then we can write the likelihood as,

.. math::
    \mathcal{L} = \prod_n p(\mathbf{x}_n) 

and as we are using a mixture of gaussians as model, we can write

.. math::
    p(\mathbf{x}_n) = \sum_k \mathcal{N} (\mathbf{x}_n | \boldsymbol\mu_k, \mathbf{\Sigma}_k) p(k)

In contrast to the K-means algorithm, the EM algorithm for Gaussian Mixture does not assign each sample to only one cluster.
Instead, it assigns each sample a set of weights representing the sample's probability of membership to each cluster.
This can be expressed with a conditional probability, :math:`p(k|n)`, given a sample :math:`n` gives the probability of the sample being drawn a certain cluster :math:`k`.
A *responsibility matrix* :math:`\mathbf{R}` with elements :math:`p_{nk}` is used to hold the probabilities.

.. math::
    p_{nk} = p(k|n) = \frac{p(\mathbf{x}_n|k)p(k)}{p(\mathbf{x}_n)}
    = \frac{\mathcal{N} (\mathbf{x}_n | \boldsymbol\mu_k, \mathbf{\Sigma}_k) p(k)}{p(\mathbf{x}_n)}

Given the data and the model parameters :math:`\boldsymbol\mu_k`, :math:`\boldsymbol\Sigma_k`, and :math:`p(k)`, we now can calculate the likeliness :math:`\mathcal{L}` and the probabilities :math:`p_{nk}`.
This this is the expectation (E) step in EM-algorithm.

In the maximization (M) step we estimate the mean, co-variances, and mixing coefficients :math:`p(k)`.
As each point has a probability of belonging to a cluster, :math:`p_{nk}`, we have to weight each sample's contribution to the paramter with that factor. The following equations are used to estimate the new set of model parameters.

.. math::
    \hat{\boldsymbol{\mu}}_k = \frac{\sum_n p_{nk}\mathbf{x}_n}{\sum_n p_{kn}}

.. math::
    \hat{\boldsymbol{\Sigma}}_k = \frac{\sum_n (\mathbf{x}_n - \hat{\mathbf{\mu}}_k)
    \otimes ( \mathbf{x}_n - \hat{\mathbf{\mu}}_k ) }
    {\sum_n p_{nk}}

.. math::
    \hat{p}(k) = \frac{1}{N}\sum_n p_{nk}

EXAMPLE OF USING EM GMM.

Expectation maximization steps with Gaussian mixtures. Only every other iteration step is plotted.

If you examine the code for the used to perform the EM algorithm on the mixture of gaussians, *em_gm*, you can see that the code uses the log-sum-exp formula \cite{numericalrecipes} to avoid underflow in the floating point calculations.
The out commented code performs the same calculations, but without the log-sum-exp formula.

..
    Problems can however still occur: ...when...

Gaussian Mixtures Regression
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is also possible to use a mixture of gaussians for regression. 

Let :math:`\mathbf{x}` and :math:`\mathbf{x}` be jointly Gaussian vectors see appendix A in [bishop2006]_.

\begin{equation}
 \left [ \begin{array}{c} \mathbf{x} \\ \mathbf{y} \end{array} \right ] \sim
 \mathcal{N} \left (
   \left [ \begin{array}{c} \mathbf{{\mu}}_x \\ \mathbf{{\mu}}_y \end{array} \right ],
   \left [ \begin{array}{cc} A & C \\ C^T & B \end{array} \right ],   
 \right )
\end{equation}

then the conditional distribution of $\mathbf{x}$ given $\mathbf{y}$ can be written as 

\begin{equation}
 \mathbf{x} | \mathbf{y} \sim
 \mathcal{N} \left ( \mathbf{{\mu}}_x + CB^{-1}(\mathbf{y}-\mathbf{\mu_{y}}), A-CB^{-1}C^T) \right )
\end{equation}

The plot in figure \ref{fig:em_gm_cond} show the conditional distribution of $\mathbf{x}$ given $\mathbf{x}$ = 0.

\begin{figure}[htb]
\begin{center}
\includegraphics[width=9cm]{figures/em_gm_cond}
\end{center}
\caption{The conditional distribution, red line, of $\mathbf{x}$ given $\mathbf{y}$ = 0.}
\label{fig:em_gm_cond}
\end{figure}

Example
^^^^^^^
This example uses EM to find the the parameters for the Gaussian Mixture Model (GMM), and then plots the conditional
distribution for one of the parameters.

.. literalinclude:: ../../examples/em_gm.py

The resulting plot should look something similar to this:

.. image:: figures/em_gm_cond_dist.png


Application Programming Interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. automodule:: pypr.clustering.gmm
   :members:

