# Building off of "Forecasting of CO2 level on Mona Loa dataset using Gaussian process regression (GPR)"

### Sofie Appel

## Motivation

The scikit tutorial, "Forecasting of CO2 level on Mona Loa dataset using Gaussian process regression (GPR)," 
uses carbon dioxide ($CO_2$) concentrations from the Mauna Loa Obseratory to predict future measures of CO2 in the atmosphere. 
Atmospheric CO2 is a direct indicator of certain climate patterns. Higher atmospheric CO2 is a direct cause of rising atmospheric temperatures.
Carbon dioxide levels increase in the atmosphere from the burning of fossil fuels. Increased atmospheric carbon dioxide 
makes it more difficult for plants to grow, which absorb CO2. Higher CO2 results in changes in climate patterns.
According to NOAA, years with higher CO2 measures have had stronger El Nino patterns, leading to extreme drought in some areas.



## Kernel Engineering

The data has multiple trends and patterns that give reason to use multiple covariance functions, 
or kernels, to capture the complexity and changes of CO2 measures in Mauna Loa. 
The CO2 measurements increase over time while also showing seasonal fluctuation and some smaller irregularities. 
Four kernels were applied before training the model to capture these trends in the data: 
radial basis function (RBF), periodic exponential sine squared, rational quadratic kernel, and white kernel. 

RBF was applied to address the 

To handle the seasonal variation, a sine squared kernel is used. 




## Limitation Model

## References

