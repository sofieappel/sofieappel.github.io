# Building off of "Forecasting of CO2 level on Mona Loa dataset using Gaussian process regression (GPR)"

### Sofie Appel

## Motivation

The scikit tutorial, "Forecasting of CO2 level on Mauna Loa dataset using Gaussian process regression (GPR)," 
uses carbon dioxide ($CO_2$) concentrations, in parts per million by volume (ppm), from the Mauna Loa Observatory to predict ($CO_2$) concentration in the years following 2001. As shown in initial exploratory data analysis, $CO_2$ concentration increased steadily from 1958 to 2001. Rising atmospheric $CO_2$ has significant consequences for the global climate. Because $CO_2$ absorbs heat, higher concentrations directly increase atmospheric temperatures through the greenhouse effect. According to NOAA, "carbon dioxide alone is responsible for about 80 percent of the total heating influence of all human-produced greenhouse gases since 1990." These rising temperatures contribute to more extreme weather, including drought, wildfires, and flooding. Many of these disasters destroy plant life, which would otherwise help absorb $CO_2$, creating a positive feedback loop that keeps more $CO_2$ in the atmosphere. It is therefore crucial to continue to observing $CO_2$ concentration patterns across the globe to understand how human activity is impacting the planet. 

In this blog post, I reproduce the results of the scikit tutorial, explain the use of kernels and the mathematical reasoning behind each, and expand the results by investigating changes in the initial parameters. 

## Kernel Engineering

A kernel creates a covariance function, typically using the distance between any two points within the domain of the data, so that a model can be fit based on the patterns revealed by the covariance function. We can use the trends of the data to help decide what kind of kernels to use. 

![co2_trend](/img/co2_trend.png)

The data has multiple trends and patterns that give reason to use multiple covariance functions, 
or kernels, to capture the complexity and changes of CO2 measures in Mauna Loa. 
The CO2 measurements increase over time while also showing seasonal fluctuation and some smaller irregularities. 
Four kernels were applied before training the model to capture these trends in the data: 
radial basis function (RBF), periodic exponential sine squared, rational quadratic kernel, and white kernel. These kernels capture multiple trends in the data including amplitude - the range of CO2 concentration in ppm - and length scale - the time scale over which the data varies. All parameters of the kernels were initially approximated then optimized by the final model.

RBF was applied to address the increase in CO2 concentration over the 43 years of the dataset. Because this kernel is only meant to capture this smooth increase, the length scale was set to 50. This tells the kernel to let the correlations stretch across the entire time frame. The output variance was also set to 50 since the range of CO2 concentration is from about 315 to 375 ppm. These both were guesses made by the author of the tutorial that are later optimized.

To handle the seasonal variation, an exponential since squared kernel is multiplied by another RBF kernel. These were multiplied to ensure that the seasonal periodicity is able to change over the almost 50 years that the data was collected over. The length scale for the RBF kernel was set to 100, double the time span of the data set, since the seasonal periodicity did not drastically change over the span of the dataset. For the exponential sine squared function, the amplitude of the seasonal variation is about 2, the period is 1 year, and the length scale used was 1, defining the waves to be moderate and not extremely peaked or flat. 

Next, a rational quadratic kernel was applied to fit the small irregularities of the data. Rational quadratic is an appropriate kernel to use for small irregularities since it is "equivalent to adding together many SE kernels with different lengthscales" (https://www.cs.toronto.edu/~duvenaud/cookbook/). 

Finally, noise was treated with an RBF kernel added to a white kernel. Noise in the data can result from irregular weather patterns at the observatory that are not necessarily repeated every year. Since these are small adjustments that need to be made, the amplitude and length scale are both small. The noise level bounds are set relatively wide as to not anchor the kernel at the initial guess. 

To create the final kernel, all four of the above kernels were added together. The characteristics, "a long term rising trend, a pronounced seasonal variation and some smaller irregularities," (tutorial) are independent of one another, calling for an additive kernel. This makes sense based on the context of the data since seasonality and small changes in weather do not depend on one another.  

The final optimized kernel is as such:

$44.8^2 * RBF(length_scale=51.6) + 2.64^2 * RBF($length_scale$=91.5) * ExpSineSquared($length_scale$=1.48, periodicity=1) + 0.536^2 * RationalQuadratic(alpha=2.89, $length_scale$=0.968) + 0.188^2 * RBF($length_scale$=0.122) + WhiteKernel($noise_level$=0.0367)$

## Limitation Model

To show the importance of using the trends in the data to set parameters, I changed the periodicity bounds to see how the model would be impacted. 

## References



