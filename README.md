# SubClonalSelection

[![Build Status](https://travis-ci.org/marcjwilliams1/SubClonalSelection.jl.svg?branch=master)](https://travis-ci.org/marcjwilliams1/SubClonalSelection.jl)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/marcjwilliams1/SubClonalSelection.jl?branch=master&svg=true)](https://ci.appveyor.com/project/marcjwilliams1/SubClonalSelection-jl/branch/master)
[![Coverage Status](https://coveralls.io/repos/github/marcjwilliams1/SubClonalSelection.jl/badge.svg?branch=master)](https://coveralls.io/github/marcjwilliams1/SubClonalSelection.jl?branch=master)
[![codecov.io](http://codecov.io/github/marcjwilliams1/SubClonalSelection.jl/coverage.svg?branch=master)](http://codecov.io/github/marcjwilliams1/SubClonalSelection.jl?branch=master)

A Julia package for inferring the strength of selection from cancer sequencing data. Package simultaneously estimates the number of subclones in the population and their relative fitnesses. This is done by generating synthetic data by simulating different population dynamics (see [CancerSeqSim.jl](https://github.com/marcjwilliams1/CancerSeqSim.jl)) and fitting this to data using Approximate Bayesian Computation (see [ApproxBayes.jl](https://github.com/marcjwilliams1/ApproxBayes.jl)).

## Getting Started
To download the package, once you're in a Julia session type the following command:
```julia
Pkg.clone("https://github.com/marcjwilliams1/SubClonalSelection.jl")
```

Then once you are in a julia session the package can be loaded with
```julia
using SubClonalSelection
```
Running `Pkg.test("SubClonalSelection")` will run a test suite to confirm the algorithm works on some test data and recovers the ground truth from some synthetic data with known input parameters.

Below is an example of how to run an analysis, for some more examples with more details go [here](https://github.com/marcjwilliams1/SubClonalSelection.jl/tree/master/example).

## Input data
Running an analysis requires variant allele frequencies (VAFs) as measured in deep sequencing of cancer samples. Generating the synthetic data assumes that the cancer is diploid, therefore any mutations falling in non-diploid regions should be removed. This does unfortunately mean that in highly aneuploid tumours, there will not be enough mutations to perform an analysis. We would recommend a minimum of 100 mutations.

## Running an analysis
The main function to perform an analysis is ```fitABCmodels```. This takes as its first argument either a vector of Floats or a string pointing to a text file with a vector of floats, which will be read in automatically. The second argument is the name of the sample you wish to analyse which will be used to write the data and plots to a file. There are then a number of keyword arguments set to reasonable defaults. More details of these arguments and their defaults can be found by typing ```?fitABCmodels``` in a Julia session.

There is some example data generated from the simulation found in the examples directory. For example the following command will run the inference on the ```oneclone.txt``` data set with 500 posterior samples and 10^6 iterations given sequencing depth of sample is 300X:
```julia
out = fitABCmodels("example/oneclone.txt",
  "oneclone",
  read_depth = 300,
  resultsdirectory = "example",
  Nmaxinf = 10^6,
  save = true,
  nparticles = 500);
```
Running this is quite computationally expensive, so running on a cluster would be recommended in most cases.

Also included are a number of functions to summarize the output and plot the posterior. ```show(out)``` will print a summary of the posterior model and parameter probabilities. We can also plot the posterior distributions.

Plot the posterior model probabilities.
```julia
plotmodelposterior(out)
```
<figure>
    <img src="https://marcjwilliams1.github.io/files/oneclone/plots/onecloneC-modelposterior-1.png" alt="modpost" width="500px">
</figure>

Plot the histogram for model 2.
```julia
plothistogram(out, 1)
```
<figure>
    <img src="https://marcjwilliams1.github.io/images/ng2018/1cloneB.png" alt="modpost" width="500px">
</figure>

Plot the posterior parameter distribution for model 2.
```julia
plotparameterposterior(out, 1)
```
<figure>
    <img src="https://marcjwilliams1.github.io/files/oneclone/plots/onecloneC-posterior-1clone-1.png" alt="modpost" width="500px">
</figure>

Note the ground truth of the parameters in this case are:

- Mutation rate: 20.0
- Number of clonal mutations: 300
- Number of subclones: 1
- Cellularity: 0.7
- Tumour population size: 10^6
- Subclone frequency: 0.58
- Fitness advantage: 1.03
- Mutations in subclone: 251
- Time emerges (tumour doublings): 9.0
- Read depth: 300X

Finally we can also save all plots and text files with posterior distributions to a directory, unless specified the default will be to create a file a directory called ```output``` in the current working directory.

```julia
saveresults(out; resultsdirectory = "example")
saveallplots(out, resultsdirectory = "example")
```
