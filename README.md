# webppl-mixture
This code is used to improve inference for mixture models in WebPPL, in cases where both the
mixture components *and* the elements are latent. While it is easy to write out such a model,
the default single-site Metropolis-Hastings algorithm is unable to make proposals which:
- Delete an arbitrary element from a mixture component, or
- Reassign an element from one component to another

This code rewrites a mixture distribution from the bottom up, first sampling elements and then
assigning them to components, and uses `factor` to rescore the trace under the correct distribution.

##  mixture(sampleParams, sampleElement, unfactor)
Samples a mixture distribution, where:
```
nComponents ~ Geometric(pComponents)
For each component:
   params = sampleParams()
   nElements ~ Geometric(pElements)
   elements = repeat(nElements, sampleElement)
```
Returns a list of components as `[{params, elements}, ...]`

Parameters:
- `sampleParams` is a thunk which samples parameters for a mixture component
- `sampleElement` is a thunk which samples an element
- if `unfactor=true`, scores for nComponents and nElements are removed from trace

All elements in all components are sampled i.i.d from sampleElement. To make
elements depend on their component's params, use factor to reweight the samples
(see example below). Use the same technique if you want a different distribution
for nComponents or nElements, e.g. a distribution with finite support. In this case
you can use `unfactor=true` to automatically subtract their scores from the trace (see example).

The function is written 'backwards', so that elements are sampled before being
assigned to mixture components. This leads to single-site Metropolis-Hastings
proposals which:
  - Add a new element
  - Remove any element
  - Reassign an element to a new mixture component
Factor statements reweight the trace to restore the semantics of the forward model
