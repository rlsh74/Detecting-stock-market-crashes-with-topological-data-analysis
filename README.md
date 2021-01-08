# Detecting Stock Market Crashes with Topological Data Analysis (TDA)

In this post we use topological data analysis (TDA) to capture these geometric changes in the time series in order to produce a reliable detector for stock market crashes. The code is implemented mainly by [`Giotto-TDA`](https://github.com/giotto-ai/giotto-tda), an open-source library for topological data analysis.

### There is little consensus about the exact definition of a financial crash

Intuitively, stock market crashes are rapid drop in asset prices. The price drop is caused by massive selling of assets, which is an attempt to close positions before prices decrease even further.

The awareness of a large speculative bubble (like in the case of the sub-primes), or a catastrophic event, will cause markets to crash. In the last two decades we have seen two major crashes: the 2000 dot-com crash and the 2008 global financial crisis.

## The results

We analyze daily prices of the S&P 500 index from 1980 to the present day. The S&P is an index that is commonly used to benchmark the state of the financial market as it measures the stock performance of the 500 large-cap US companies.

This highlights one of the key motivations behind TDA, namely that topology and geometry can provide a powerful method to abstract subtle structure in complex data.

![fig1](/images/fig1.png)
[Detection of stock market crashes from baseline (left) and topological (right) models]

Let's describe in more detail the two approaches.

### A simple baseline

Given that market crashes represent a sudden decline of stock prices, one simple approach to detect these changes involves tracking the first derivative of average price values over a rolling window. Indeed, in the figure below we can see that this naïve approach already captures the Black Monday crash (1987), the burst of the dot-com bubble (2000–2004), and the financial crisis (2007–2008).

![fig2](/images/fig2.png)
[Magnitude of the first derivative of mean close prices between successive windows]

By normalizing this time series to take values in the [0,1] interval, we can apply a threshold to label points on our original time series where a crash occurred.

![fig3](/images/fig3.png)
[Crash probability for baseline model (left), with points above threshold shown on original time series (right)]

> *Evidently this simple method is rather noisy*

With many points labeled as a crash, following this advice will result in over-panicking and selling your assets too soon. Let's see if TDA can help us reduce the noise in the signal and obtain a more robust detector!

## The TDA Pipeline

The mathematics underlying TDA is deep and won't be covered in this project; but for our purposes, it is sufficient to think of TDA as a means to extract informative features which can be used for modeling downstream.

The pipeline consists in: 2) embedding the time series into a point cloud and constructing sliding windows of point clouds, 3) building a filtration on each window to have an evolving structure encoding the geometrical shape of each window, 4) extracting the relevant features of those windows using persistence homology, 5) comparing each window by measuring the difference of those features from one window to the next, 6) constructing an indicator of crash based on this difference.

![pipeline](/images/pipeline.JPG)

### Time series as point clouds - Takens' Embedding

A typical starting point in a TDA pipeline is to generate a simplicial complex from a point cloud. Thus, the crucial question in time series applications is how to generate such point clouds? Discrete time series, like the ones we are considering, are typically visualised as scatter plots in two dimensions. This representation makes the *local* behaviour of the time series easy to track by scanning the plot from left to right. But it is often ineffective at conveying important effects which may be occurring over larger time scales.

One well-known set of techniques for capturing periodic behaviour comes from Fourier analysis. For instance, the [discrete Fourier transform](https://en.wikipedia.org/wiki/Discrete_Fourier_Transform) of a temporal window over the time series gives information on whether the signal in that window arises as the sum of a few simple periodic signals.

For our purposes we consider a different way of encoding a time-evolving process. It is based on the idea that some key properties of the dynamics can be unveiled effectively in higher dimensions. We begin by illustrating a way of representing a univariate time series as a *point cloud*, i.e. a set of vectors in a Euclidean space of arbitrary dimension.

The procedure works as follows: we pick two integers *d* and *τ*. For each time *tᵢ ∈ (t₀, t₁, …)*, we collect the values of the variable *y* at *d* distinct times, evenly spaced by *τ* and starting at *tᵢ*, and present them as a vector with *d* entries, namely:

![equation](https://latex.codecogs.com/svg.latex?%5Clarge%20Y_%7Bt_i%7D%20%3D%20%28y_%7Bt_i%7D%2Cy_%7Bt_i&plus;%5Ctau%7D%2C%5Cdots%2Cy_%7Bt_i&plus;%28d-1%29%5Ctau%7D%29)

The result is a set of vectors in *d*-dimensional space! *τ* is called the time delay parameter, and *d* the embedding dimension.

This time-delay embedding technique is also called Takens' embedding after Floris Takens, who demonstrated its significance with a celebrated [theorem](https://en.wikipedia.org/wiki/Takens%27s_theorem) in the context of nonlinear dynamical systems.

Finally, applying this procedure separately on sliding windows over the full time series leads to a ***time series of point clouds*** (one per sliding window) with possibly interesting topologies. The GIF below shows how such a point cloud is generated in 2-dimensions.

![taken](/images/takens.gif)

### From point clouds to persistence diagrams

Now that we know how to generate a time series of point clouds, what can we do with this information? Enter persistent homology, which looks for topological features in a simplicial complex that persist over some range of parameter values. Typically, a feature, such as a hole, will initially not be observed, then will appear, and after a range of values of the parameter will disappear again.

![fig4](/images/fig4.png)
![fig5](/images/fig5.png)

### Distances between persistent diagrams

Given two windows and their corresponding persistence diagrams, we can calculate a variety of distance metrics. Here we compare two distances, one based on the notion of a persistence landscape, the other on Betti curves.

![fig6](/images/fig6.png)
[Magnitude of the landscape (left) and Betti curve (right) distances between successive windows]

From these figures we can infer that the metric based on landscape distance is less noisy than the Betti curves.

### A topological indicator

Using the landscape distance between windows as our topological feature, it is a simple matter to normalize it as we did for the baseline model. Below we show the resulting detection of stock market crashes for the dot-com bubble and global financial crisis. Compared to our simple baseline, we can see that using topological features appears to reduce the noise in the signal of interest.

![fig7](/images/fig7.png)
![fig8](/images/fig8.png)
[Crash probabilities and detections using topological features. The time ranges correspond to the dot-com bubble in 2000 (upper) and the global financial crisis in 2008 (lower)]

## Conclusion

Our results suggest that the periods of high volatility preceding a crash produce geometric signatures that can be more robustly detected using topological data analysis. However, these results concern only a specific market and for a short period of time, so one should further investigate the robustness of the procedure on different markets and varying thresholds. Nevertheless, the results are encouraging and open up some interesting ideas for future development!

