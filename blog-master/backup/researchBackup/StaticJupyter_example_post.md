+++
title = "A Static Jupyter Notebook Example"
date = '2017-12-12'
tags = [ "Deep Learning", "Neural Networks", "Class17/18",]
categories = ["research"]
banner = "img/banners/deepLearningKlein.png"
author = "Class of Winter Term 2017 / 2018"
disqusShortname = "https-wisample-github-io-blog"
description = "This is an example of a jupyter notebook to give you an idea of how your blog post could look like."
+++

This is an example of a jupyter notebook to give you an idea of how your blog post could look like.





## Python Analysis I

```python
# %load std_ipython_import.txt
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import pymc3 as pm

```


#### Model


```python
# Using the Theano @as_op decorator with a custom function to calculate the threshold probabilities.
# Theano cannot compute a gradient for these custom functions, so it is not possible to use
# gradient based samplers in PyMC3.
# http://pymc-devs.github.io/pymc3/notebooks/getting_started.html#Arbitrary-deterministics
@as_op(itypes=[tt.dvector, tt.dscalar, tt.dscalar], otypes=[tt.dvector])
def outcome_probabilities(theta, mu, sigma):
    out = np.empty(nYlevels)
    n = norm(loc=mu, scale=sigma)
    out[0] = n.cdf(theta[0])
    out[1] = np.max([0, n.cdf(theta[1]) - n.cdf(theta[0])])
    out[2] = np.max([0, n.cdf(theta[2]) - n.cdf(theta[1])])
    out[3] = np.max([0, n.cdf(theta[3]) - n.cdf(theta[2])])
    out[4] = np.max([0, n.cdf(theta[4]) - n.cdf(theta[3])])
    out[5] = np.max([0, n.cdf(theta[5]) - n.cdf(theta[4])])
    out[6] = 1 - n.cdf(theta[5])
    return out

with pm.Model() as ordinal_model_single:

    theta = pm.Normal('theta', mu=thresh, tau=np.repeat(.5**2, len(thresh)),
                      shape=len(thresh), observed=thresh_obs, testval=thresh[1:-1])

    mu = pm.Normal('mu', mu=nYlevels/2.0, tau=1.0/(nYlevels**2))
    sigma = pm.Uniform('sigma', nYlevels/1000.0, nYlevels*10.0)

    pr = outcome_probabilities(theta, mu, sigma)

    y = pm.Categorical('y', pr, observed=df.Y.cat.codes.as_matrix())
```


```python
with ordinal_model_single:
    step = pm.Metropolis([theta, mu, sigma, pr, y])
    trace1 = pm.sample(10000, step)
```

    100%|██████████| 10500/10500 [02:22<00:00, 73.63it/s]



```python
pm.traceplot(trace1);
```

An alternative way is to post the jupyter notebook on Gist.

<script src="https://gist.github.com/HsueanL/fe837b828a814f70967e9dd485912d67.js"></script>


Besides demonstrating code, it makes sense to show the reader plots and outputs.

![png](/blog/img/chapter23/output_11_0.png)
