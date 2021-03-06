# GDP Layer

## Datasets

### Nightlight Imagery

Visible Infrared Imaging Radiometer Suite (VIIRS) Day/Night Band (DNB) Annual Composite for 2015 that removes outliers occurring due to fires and other ephemeral lights, and zeroes out background (non-light) pixels. Each pixel expressed as nanoWatts/cm<sup>2</sup>/steradian

**Source:** _SVDNB\_npp\_20150101-20151231\_00N060E\_vcm-orm-ntl\_v10\_c201701311200.avg\_rade9.tif_ from [https://bit.ly/2tG5vtg](https://bit.ly/2tG5vtg)

### Gas Flares

A fantastic [paper](http://www.mdpi.com/2072-4292/5/9/4423) provides for a method to detect gas flares. The raw datasets provided by the National Centers for Environmental Information (NCEI) are [here](https://ngdc.noaa.gov/eog/viirs/download_viirs_fire.html). This was then cleaned by SkyTruth:

*SkyTruth starts with this data and then cleanses, clusters and highlights to produce the visualization seen in the flaring map, giving the original dataset a new power.*

This SkyTruth [dataset](https://www.skytruth.org/viirs/) was used in the GDP analysis.

### Population

Global Human Settlement Population Grid for 2015. Each pixel expressed as absolute number of people

**Source:** _GHS\_POP\_GPW42015\_GLOBE\_R2015A\_54009\_1k_ from https://bit.ly/2lGWM5K

### Gross State Domestic Product (GSDP) &amp; Share of Agriculture in GSDP

GSDP (at Market Price) for 2015-2016 at constant 2011-2012 prices

**Sources**

For all states and territories except West Bengal, the GDP numbers were readily available at [http://mospi.gov.in/sites/default/files/press\_releases\_statements/StatewiseDomesticProduct\_28feb18.xls](http://mospi.gov.in/sites/default/files/press_releases_statements/StatewiseDomesticProduct_28feb18.xls) .

Since the Government of West Bengal hasn&#39;t released the GDP figures for 2011-2012 prices yet, a proxy was generated. Steps below:

1. West Bengal nominal GDP figures (only till 2014-2015) were retrieved from [http://niti.gov.in/content/2004-05-series](http://niti.gov.in/content/2004-05-series)
2. 2015-2016 nominal GDP figures were then estimated by taking an average growth rate of 14.41% in nominal GDP using the growth rate in previous years
3. 2015-2016 GDP at 2011-2012 prices was then estimated using the GDP deflator in [https://data.worldbank.org/indicator/NY.GDP.DEFL.KD.ZG](https://data.worldbank.org/indicator/NY.GDP.DEFL.KD.ZG)
4. Agriculture&#39;s share in GDP was calculated in a similar way as in 2.


### State Boundaries

[https://github.com/datameet/maps/tree/master/Districts](https://github.com/datameet/maps/tree/master/Districts)



## Initial Cleansing

Initially, areas with extremely bright lights got allocated commensurate GDP figures – for example visible spikes were seen in places like Jamnagar (Gujarat), Surasaniyanam (slightly East of Vijayawada in Andhra Pradesh) and in Dibrugarh (Assam). On deeper inspection, it seemed like this was largely due to the prevalence of gas flaring activities in these regions. 

Using the Skytruth dataset, gas flaring areas were plotted, buffered by a fixed distance and then this buffered area was used to zero out the original radiance value shown by the untouched VIIRS dataset. In addition, all pixels emitting a radiance of above 160 nanoWatts/cm<sup>2</sup>/steradian were also zeroed out on the assumption that these were undetected gas flares (see [caveats](https://www.skytruth.org/viirs/) of the SkyTruth dataset). A value of 160 was picked based on some simple localised analyses that showed that the brightest regions in cities do not emit more than around 100 nanoWatts and that the brightest airports don't emit more than around 150 nanoWatts.


## Methodology

The methodology draws heavily from this [paper](https://www.researchgate.net/publication/228371381_Shedding_Light_on_the_Global_Distribution_of_Economic_Activity)

1. Add the light value of all pixels within each region i. This gives the sum of lights for each region (SLi)
2. Derive Ri = SLi/GSDPi where GSDPi represents the gross state domestic product for region i and Ri is a ratio that essentially tells us &#39;how much light is emitted as a proportion of the GSDP&#39; for each region i.
3. The GSDP for each pixel is finally calculated as follows:
a)Industrial and Non-Agricultural Component: Calculate how much light is emitted by that pixel relative to the total light emitted by the state – say 2%. Multiply 2% by the SGDP for the state giving SGDPa
b)Agricultural/Non-Industrial Component: Calculate how many people live in the pixel relative to the total population of the state – say 5%. Multiply 5% by the agricultural SGDP for the state giving SGDPb.
c)Add SGDPa and SGDPb


## Final Result

The final result is a 1km<sup>2</sup> grid depicting 2015-2016 GDP at constant 2011-2012 prices. Each pixel expressed as millions of INR.


## Limitations

Since state-level GDP data is collected by the respective State Governments (and not the Union Government), the GDP methodology might not be standardised across the states. While this would not affect within-state comparisons of GDP, it could affect across-state comparisons.

Zeroing out gas flare pixels might not be optimal since a GDP of 0 is then assigned to these pixels. That being said, the error is definitely minimised.

In essence, the limitations of the methodology in Ghosh, T., Powell, R., Elvidge, C. D., Baugh, K. E., Sutton, P. C., &amp; Anderson, S. (2010) apply to this data-set as well. That leads us to the important caveat that while the original intention of the methodology was to disaggregate GDP, the dataset might work much better if looked at as a bunch of clusters rather than as a bunch of individual grids.


# Nightlight Development Index (NLDI) Layer

The NLDI is, very simply, an absolute measure of how evenly nightlight is distributed across populations. The genesis of this index is in this [paper](http://www.soc-geogr.net/7/23/2012/sg-7-23-2012.pdf). The methodology used in our tool is similar to the one used in the original paper which itself is heavily influenced by the calculation of the [Gini index] (https://en.wikipedia.org/wiki/Gini_coefficient#Calculation) - a measure of income equality. However since there are a few differences, we provide a brief summary of the methodology here.

For each unique district, the NLDI is calculated as follows:

1. For each pixel, derive 'radiance per capita'. This is nothing but the VIIRS value (The original paper uses DMSP data) for that pixel divided by the population in that pixel. More intuitively, this is the amount of light each person gets in a 1km<sup>2</sup> area.
2. If you have a district of 100km<sup>2</sup>, that equates to 100 pixels meaning you'll have 100 values at the end of step 1.
3. Filter any duplicate values and sort these values in ascending order.
4. Once sorted, for each value (radiance per capita):
    1. calculate the total population living with that value. Basically take every pixel that yields the given value and sum the population across these pixels
    2. calculate the total light emitted by pixels that have the given value. Basically take every pixel that yields the given value and sum the light across these pixels
5. Now calculate cumulative percentages for total population and total light. 
6. Create a graph with cumulative total light being the Y-axis and cumulative total population being the X-axis.
7. Calculate the NLDI index as the area under the perfect equality triangle minus the area under the curve in 6.

# Airlinks Layer

The Airlinks Layer represents total weekly passenger capacity on all domestic routes based on the Summer Schedule (25th March 2018 to 27th October 2018) [released](http://dgca.nic.in/dom_flt_schedule/flt_index.htm) by the DGCA. Raw data and Python scripts are [here](https://github.com/wireman27/india_gdp_new/tree/master/airlinks).

(Thanks to Ben Balter's sleek [Word to Markdown tool](https://word-to-markdown.herokuapp.com/))


