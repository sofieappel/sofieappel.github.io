# Climate Change in Glacier National Park
Sofie Appel

<img
    src = "/img/glacier.jpg"
    alt = "me"
    width = "50%">
    
**Figure 1:** Glacier National Park, Montana, United States of America.

Glacier National Park has seen significant changes in its landscape since its establishment. 
The national park in northern Montana had 150 active glaciers in 1850. Now, there are only 25 (NASA). 
These changes have significant implications for the wildlife and human populations that rely on the meltwater from glaciers. 
While glaciers tend to naturally fluctuate based on the climate, consistent shrinkage of the glaciers is evidence of the impact of rising temperatures due to human caused climate change. 
To investigate the changing climate in Glacier National Park, a linear regression analysis was performed on the average annual maximum temperatures in Glacier.

## Data Preprocessing

The dataset used in this project was acquired using an API call of NOAA Climate Data Online. 
The data came from the Kalispell Glacier Airport, which is slightly southwest of the actual park. 
This station has maximum and minimum daily temperature data available from 1896 to 2026 but only seven years of average daily temperature recorded. 
Therefore, maximum daily temperature data was used for this project to observe how the high temperatures have changed since the start of this dataset. 

The original dataset had temperatures in the units Fahrenheit. The final dataframe transformed the temperature in Fahrenheit to Celsius. 

To better understand the change in climate year over year and reduce seasonal variation, the daily temperature data was transformed 
from daily observations to yearly averages. The years 1896 and 2026 were then removed because the observations did not cover the full year. 

<p align="center">
<img
    src = "/img/glacier_df.png"
    alt = "me"
    width = "50%">
</p>

**Figure 2:** Left - Initial dataframe with maximum daily temperature observations. Right - Transformed dataframe with annual average maximum temperature observations in units Fahrenheit and Celsius. 

When plotted, the average annual maximum temperatures fluctuate between about 10 and 14 degrees Celsius, with a slight overall increase over the century.

<embed type="text/html" src="./annual_temp_glacier.html" width="600" height="600">

**Figure 3:** Interactive plot of average annual maximum temperatures in Glacier National Park.

## Linear Regression Results

When ordinary least squares linear regression was applied to the data, the trend showed an average increase in maximum temperature of 0.0122 degrees Celsius per year. 
This means the average annual maximum temperatures have increased by about 1.5 degrees Celsius, leading to the major shrinkage of glaciers in the park.

<p align="center">
<img
    src = "/img/glacer_trend.png"
    alt = "lin_reg_glacier"
    width = "75%">
</p>

**Figure 4:** The average annual maximum temperatures in Glacier have increased since 1897 and have done so at a rate of approximately 0.0122 degrees Celsius per year.

## References

NASA. (2026, March 5). *World of Change: Ice Loss in Glacier National Park.* https://science.nasa.gov/earth/earth-observatory/world-of-change/glacier-national-park/
