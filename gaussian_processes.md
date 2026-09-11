# Building off of "Forecasting of CO2 level on Mona Loa dataset using Gaussian process regression (GPR)"

### Sofie Appel

## Motivation

The scikit tutorial, "Forecasting of CO2 level on Mauna Loa dataset using Gaussian process regression (GPR)," 
uses carbon dioxide ($CO_2$) concentrations, in parts per million by volume (ppm), from the Mauna Loa Observatory to predict $CO_2$ concentration in the years following 2001. As shown in initial exploratory data analysis, $CO_2$ concentration increased steadily from 1958 to 2001. Rising atmospheric $CO_2$ has significant consequences for the global climate. Because $CO_2$ absorbs heat, higher concentrations directly increase atmospheric temperatures through the greenhouse effect. According to NOAA, "carbon dioxide alone is responsible for about 80 percent of the total heating influence of all human-produced greenhouse gases since 1990" (Lindsey). These rising temperatures contribute to more extreme weather, including drought, wildfires, and flooding. Many of these disasters destroy plant life, which would otherwise help absorb $CO_2$, creating a positive feedback loop that keeps more $CO_2$ in the atmosphere. It is therefore crucial to continue to observe $CO_2$ concentration patterns across the globe to understand how human activity is impacting the planet. 

In this blog post, I reproduce the results of the scikit tutorial, explain the use of kernels and the mathematical reasoning behind each, and expand the results by investigating changes in the initial parameters. 

## Kernel Engineering

A kernel creates a covariance function, typically using the distance between any two points within the domain of the data, so that a model can be fit based on the patterns revealed by the covariance function. We can use the trends of the data to help decide what kind of kernels to use. 

![co2_trend](/img/co2_trend.png)
*Figure 1: CO2 concentrations at the Mauna Loa Observatory with trend line*

The data has multiple trends and patterns that give reason to use multiple covariance functions, 
or kernels, to capture the complexity and changes of $CO_2$ measures in Mauna Loa. 
The $CO_2$ measurements increase over time while also showing seasonal fluctuation and some smaller irregularities. 
Four kernels were applied before training the model to capture these trends in the data: 
radial basis function (RBF), periodic exponential sine squared, rational quadratic kernel, and white kernel. These kernels capture multiple trends in the data including amplitude - the range of $CO_2$ concentration in ppm - and length scale - the time scale over which the data varies. All parameters of the kernels were initially approximated then optimized by the final model.

RBF was applied to address the increase in $CO_2$ concentration over the 43 years of the dataset. Because this kernel is only meant to capture this smooth increase, the length scale was set to 50. This tells the kernel to let the correlations stretch across the entire time frame. The output variance was also set to 50 since the range of $CO_2$ concentration is from about 315 to 375 ppm. 

To handle the seasonal variation, an exponential sine squared kernel is multiplied by another RBF kernel. These were multiplied to ensure that the seasonal periodicity is able to vary over the almost 50 years that the data was collected over. The length scale for the RBF kernel was set to 100, double the time span of the data set, since the seasonal periodicity did not drastically change over the span of the dataset. For the exponential sine squared function, the amplitude of the seasonal variation was set to 2, the period is 1 year, and the length scale used was 1, defining the waves to be moderate and not extremely peaked or flat. This case is proven mathematically as follows:

The exponential sine squared kernel is in the form: 

$$k(x,x')=a^2 \exp\left(-\frac{2\sin^2(\pi|x-x'|/p}{l^2})\right)$$

where a is the amplitude, p is the period, and l is the length scale. This function is purely periodic and stays the same as $x-x'$, the distance between two arbitrary points, increases:

![seasonal plot](/img/seasonal.png)
*Figure 2: Exponential sine squared function where x is the lag between two points, showing a solely periodic trend*

This function is bounded between $a^2e^{-2/l^2}$ and $a^2$ since $sin^2$ is bounded between 0 and 1. This means that the model would treat two points that are 50 years apart just as it would for two points that are 1 year apart. Multiplying by the RBF kernel fixes this. 

The RBF kernel is in the form:

$$k(x,x')= \exp\left(-\frac{(x-x')^2}{2l^2}\right)$$

This function is $\leq 1$ for all x, meaning it will act as a shrinking factor to the exponential sine squared kernel as the difference between x and x' increases. For example, for a 43 year span of points, the correlation shrinking factor is:

$$exp\left(-\frac{43^2}{2(100)^2}\right) \approx 0.91$$

while over a 100 year span the shrinking factor becomes: 

$$exp\left(-\frac{100^2}{2(100)^2}\right) \approx 0.61$$

This reduces the covariance between observations that are far apart in time, allowing the seasonal pattern to change gradually rather than requiring the same seasonal behavior indefinitely. Keeping the length scale large, however, makes this only a slight change. It is almost unobservable on our time scale of 43 years of observations. When expanded to look at a time span of 100 years, the difference is more noticeable: 

![comparison](/img/comparison_plot2.png)
*Figure 3: Comparison of exponential sine squared kernel with RBF x ExpSinSquared*

Next, a rational quadratic kernel was applied to fit the small irregularities of the data. Rational quadratic is an appropriate kernel to use for small irregularities since it is "equivalent to adding together many SE kernels with different lengthscales" (Duvenaud). Irregularities in the data occur at multiple, unpredictable timescales, so a kernel that has a mixture of many length-scales is a better fit than a single RBF.

Finally, noise was treated with an RBF kernel added to a white kernel. Noise in the data can result from irregular weather patterns at the observatory that are not necessarily repeated every year. Since these are small adjustments that need to be made, the amplitude and length scale are both small. The noise level bounds are set relatively wide as to not anchor the kernel at the initial guess. 

To create the final kernel, all four of the above kernels were added together. The characteristics, "a long term rising trend, a pronounced seasonal variation and some smaller irregularities," (scikit learn) are independent of one another, calling for an additive kernel. This is intuitive since seasonality and small changes in weather do not depend on one another.  

The final, optimized kernel is as such:

$44.8^2 * RBF($ length_scale $=51.6) + 2.64^2 * RBF($ length_scale $=91.5) * ExpSineSquared($ length_scale $=1.48, periodicity=1) + 0.536^2 * RationalQuadratic(alpha=2.89,$ length_scale $=0.968) + 0.188^2 * RBF($ length_scale $=0.122) + WhiteKernel($ noise_level $=0.0367)$

This model shows that the data has the following characteristics:
- The long term trend has a length scale of 51.6 years, close to the 43 year time span of the data
- The long term trend has an amplitude of 44.8 ppm, indicating that the trend component can produce relatively large changes in predicted $CO_2$ concentration.
- The seasonal length scale is 1.48, meaning the seasonal trend is relatively smooth
- The seasonal amplitude is 2.64, meaning over a year, $CO_2$ concentration can vary by about 5.28 ppm
- The irregularities component has a length scale of 0.968 and a large alpha, meaning it captures small deviations that do not persist long
- The white noise level is 0.0367, suggesting that there is very little unexplained randomness

## Limitation Model

To investigate how hyperparameter tuning changes extrapolation in Gaussian Process Regression, I changed the periodicity bound from fixed for the original model to a range of 0.9 to 1.1. A comparison of the models is shown below: 

![period_bound](/img/period_bound.png)
*Figure 4: Comparison of extrapolation models when periodicity bound is fixed versus loose*

The loosened periodicity model predicts substantially lower future $CO_2$ levels. Its optimized kernel showed significant differences from the original: 

$14.6^2 * RBF($ length_scale $=97.7) + 0.133^2 * RBF($ length_scale $=108) * ExpSineSquared($ length_scale $=1.39, periodicity=1) + 0.0456^2 * RationalQuadratic(alpha=0.322,$ length_scale $=1.34) + 0.0108^2 * RBF($ length_scale $=0.117) + WhiteKernel($ noise_level $=0.000124)

This change was intended to test only the seasonal kernel's flexibility; however, it caused the joint hyperparameter optimization to converge to a completely different solution. The long term trend's amplitude dropped from 44.8 to 14.6, the seasonal amplitude dropped from 2.64 to 0.133, and the noise level dropped from 0.0367 to 0.000124. These significant changes in the final model demonstrate that GP hyperparameters are not independently optimized. 


## References

Duvenaud, D. *The Kernel Cookbook: Advice on Covariance functions*. https://www.cs.toronto.edu/~duvenaud/cookbook/

scikit learn. *Forecasting of CO2 level on Mona Loa dataset using Gaussian process regression (GPR)*. https://scikit-learn.org/stable/auto_examples/gaussian_process/plot_gpr_co2.html

Lindsey, R. (2025, May 21). *Climate change: atmospheric carbon dioxide*. NOAA. https://www.climate.gov/news-features/understanding-climate/climate-change-atmospheric-carbon-dioxide




