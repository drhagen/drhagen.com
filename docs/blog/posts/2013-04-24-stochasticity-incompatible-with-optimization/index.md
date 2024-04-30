---
title: "Stochasticity Incompatible with Optimization"
date: 2013-04-24
categories:
  - "biology"
---

I have been thinking about the design of a framework capable of modelling an entire cell. Two of the design specifications, stochasticity and optimization, that I have been planning to incorporate seem to be incompatible.

<!-- more -->

Stochasticity is a form of randomness in cells that originates from the fact cells are made of molecules and these molecules react after bouncing around until they hit something that makes them react. This makes the concentration of the molecules and, hence, the cell change somewhat  randomly. Only when there are large numbers of each molecular species, these random fluctuations average out so that overall behavior is quite smooth and predictable.

Optimization is the automated changing of the parameters to make the model more consistent with measured data. I usually envision a black box into which data goes in and a description of the parameter space consistent with that data comes out. The only work the researcher has to put in to define the likelihood function for new types of data.

For mass action models for biological systems, I have found that this optimization fairly easy to accomplish for large models. There is usually one minimum or, occasionally, a small number of local minima.  The minimum can be found efficiently by sequential quadratic programming using an analytical gradient, once the problem has been properly scaled and a few other tweaks for biological systems have been applied. Starting from the best-fit parameters, the model ensemble can be characterized effectively by a linear expansion or by Hastings sampling of the likelihood. From here, one can compute estimates and uncertainties on the topology, parameters, and predictions. But the key piece is that analytical gradient! All my experience has shown that, without that, an algorithm cannot figure out which way to move in parameter space to get to the better parameters; as the number of parameters increases, the parameter space grows exponentially until it is quickly a hyperdimensional labyrinth.

Optimization becomes a problem when considering stochasticity. And stochasticity cannot be ignored if we hope to make an accurate model of a simple living organism because the assumption of a large number of every species is simply not true for many species. If we use a deterministic approximation to the stochastic noise, such as the linear noise approximation (LNA) or mass fluctuation kinetics (MFK), I suspect that optimization would still work well. It is tricky to compute the gradient on LNA or MFK, but I have done it. Nevertheless, I am skeptical that noise approximations can properly capture all behaviors of the cell. In particular, DNA is a reactant that exists and is important in only 0, 1, or 2 copies per cell. Ribosome assembly is also problematic because the number of states is astronomically larger than the number of ribosome copies. If we use a stochastic model to represent these processes, then we will have a problem for several reasons: (1) there is no good way to compute the likelihood for some stochastic data, and (2) there is no way to compute the derivative of the likelihood with respect to the parameters for any stochastic data.

How could you not compute the likelihood? you may ask. Consider some single cell data where a batch of cells had many proteins observed over time. The naive way to compute the likelihood that a particular parameter set is true is to simulate the stochastic model many times and try to match each simulation to each data set, where each data set comes from the observation of a single cell. That is, ask what is the probability that I would see this data set given that the cell had these stochastic trajectories. Then multiply those likelihoods across the data sets for each simulation and average across all simulations—this is the likelihood that this data would be seen given this parameter set. This is not so bad unless the measurement uncertainty is smaller than the stochastic uncertainty. Then, the odds that a data point would be consistent with the trajectories of a simulation is quite small even if the perfect parameters were used. Even when using perfect parameters, a simulation lining up with a data point will be fairly rare, making it very expensive to see enough instances to get a good estimate of how often this happens.  This is still not so bad until the large number of data points per experiment is considered. If enough measurements are taken per cell in this experiment, the odds that all measurements will line up with a particular trajectory is infinitesimal. As a result, all trajectories will come back with essentially 0 probability for all simulations for all parameters unless a enormously impractical number of simulations are done.

One way around this problem is to decouple the data points in a single cell from each other, simply multiply the likelihood over all data points rather than all data sets. The fraction of simulations that match a particular data point will be a lot higher than the fraction that match an entire data set. But this is not entirely correct, this throws away the correlation information between the data points. For example, if the data measured one state at the beginning and end of a simulation and the true model would either be high, low, rise from low to high, or fall from high to low, then a model that would be only either low or high only would fit just as well as the true model. Fortunately, it is not completely wrong to throw out information. The parameter estimates will have higher uncertainty because some parameter sets that are clearly wrong according to the data will be retained, but no correct parameter sets will be removed. If it is easier to collect more data than to fit the data we already have, this may be the right course of action.

The gradient, the derivative of the likelihood with respect to the parameters, is a critical part of gradient-descent-based global optimizers and is even more problematic for stochastic models. Assuming that the likelihoods can be calculated precisely, then the gradient can be calculated by finite differences. But as far as I can imagine, the likelihood will always be random with a stochastic model, meaning that taking the difference between two nearby points in parameter space will be dominated by the random noise, especially since the parameter space of biological models is shaped like thin needles with a very gentle slope toward the better parameters. Without a precise analytical gradient, it is extremely difficult to fit a biological model with many parameters. How one would construct an analytical gradient for a model that is not analytical eludes me. One possibility would be to fit the deterministic model to the data, and hope it gets close enough before using the stochastic model for predictions. Another possibility would be dump optimization entirely and require the system parameters to be determined outside the cell in a macroscopic context. In a any case, it seems unlikely that I will be able to build my black box to fit whole-cell stochastic models to huge amounts of single-cell data.