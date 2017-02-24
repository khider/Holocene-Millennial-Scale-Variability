
<img src="http://www.organicdatacuration.org/linkedearth/images/5/51/EarthLinked_Banner_blue_NoShadow.jpg">
# Testing the Millennial-Scale Holocene Solar-Climate Connection in the Indo-Pacific Warm Pool

<p>D. Khider<sup>1</sup>, J. Emile-Geay<sup>1</sup>, N. McKay <sup>2</sup>, C.S. Jackson <sup>3</sup>, C. Rouston <sup>2</sup></p>

<p><sup>1</sup> University of Southern California<br>
<sup>2</sup> Northern Arizona University<br>
<sup>3</sup> The University of Texas at Austin</p>

## Motivation

<p>The existence of 1000 and 2500-year periodicities found in reconstructions of total solar irradiance (TSI) and a number of Holocene climate records has led to the hypothesis of a causal relationship<sup>[1-3]</sup>. However, attributing millennial-scale variability to solar focring requires a mechanim by which small changes in total solar irradiance can influence a global climate response. One possible amplifier within the climate system is the ocean. If this is the case, then we need to know about where and how this may be occuring. On the other hand, the similarity in spectral peaks could be merely coincidental, and this should be made apparent by a lacl of coherence in how that power and phasing are distributed in time and space.</p>

<p style="font-weight: bold"> The plausibility of the solar forcing hypothesis is assessed trough a Bayesian model of the age uncertainties affecting marine sedimentary records that is propagated through spectral analysis of the climate and forcing signals at key frequencies<sup>[4]</sup>.</p>

## Methods

### Data Selection

All the data used in this study are availabled on the <a href="http://wiki.linked.earth"> LinkedEarth wiki </a>, which supports complex queries. The three criteria used here are:
<ol>
    <li> SST-sensitive proxies, including Mg/Ca, U<sub>37</sub><sup>k'</sup>, TEX86 </li>
    <li> >5000 years long </li>
    <li> Radiocarbon-based chronologies </li>
</ol>

<p>The LinkedEarth wiki can be queried using a variety a methods detailed <a href="http://wiki.linked.earth/Querying_the_Datasets#Querying_Linked_Earth_Data_from_another_Program.2FScript"> here </a>.</p>


```python
%%python
import json
import requests

url = "http://wiki.linked.earth/store/ds/query"

query = """prefix xsd: <http://www.w3.org/2001/XMLSchema#>
#Distinct gives you a single value for the variables you are asking for
SELECT distinct ?dataset
WHERE {
  
  ?dataset <http://linked.earth/ontology#includesPaleoData> ?d.
  ?d <http://linked.earth/ontology#foundInMeasurementTable> ?t.
  ?t <http://linked.earth/ontology#includesVariable> ?v.
  ?t <http://linked.earth/ontology#includesVariable> ?v1.
  OPTIONAL {?v <http://linked.earth/ontology#measuredOn> ?a.
  ?a a <http://linked.earth/ontology#MarineSediment>.}
  #variable ?v of the dataset should be about either Mg/ca, Uk37 or TEX86
  VALUES ?proxy { "Mg/Ca" "Uk37" "TEX86"}
  ?v <http://linked.earth/ontology#onProxyObservationProperty> ?p.
  ?p <http://www.w3.org/2000/01/rdf-schema#label> ?proxy.
  
  #Variable ?v1 should be about Age and pass the following filters.
  ?v1 ?pr ?i.
  ?pr <http://www.w3.org/2000/01/rdf-schema#label> "OnInferredVariableProperty".#Trick, this should be URI
  ?i <http://www.w3.org/2000/01/rdf-schema#label> "Age".#Trick, this should be URI
  {
  ?v1 <http://linked.earth/ontology#hasUnits> "yr BP".
  ?v1 ?hasMinValue ?e1.
  ?hasMinValue <http://www.w3.org/2000/01/rdf-schema#label> "HasMinValue".#trick
  ?v1 ?hasMaxValue ?e2.
  ?hasMaxValue <http://www.w3.org/2000/01/rdf-schema#label> "HasMaxValue". #trick
   
  filter(?e1<5000 && ?e2>7000 && abs(?e2-?e1)>5000).
  }
  UNION
  {
    ?v1 <http://linked.earth/ontology#hasUnits> "kyr BP".
  ?v1 ?hasMinValue ?e1.
  ?hasMinValue <http://www.w3.org/2000/01/rdf-schema#label> "HasMinValue".#trick
  ?v1 ?hasMaxValue ?e2.
  ?hasMaxValue <http://www.w3.org/2000/01/rdf-schema#label> "HasMaxValue". #trick
   
  filter(?e1<5 && ?e2>7 && abs(?e2-?e1)>5).
  }
  
  # Now create a filter for the Indo-Pacific warm pool (longitude: 100 to 160E and latitude: 30S to 30N)
  ?dataset <http://linked.earth/ontology#collectedFrom> ?z.
  ?z <http://www.w3.org/2003/01/geo/wgs84_pos#lat> ?lat. # Get the latitude property
  filter(xsd:float(?lat)<30 && xsd:float(?lat)>-30). #filter
  ?z <http://www.w3.org/2003/01/geo/wgs84_pos#long> ?long. # Get the longitude property
  filter(xsd:float(?long)<160 && xsd:float(?long)>100). #filter
}"""

response = requests.post(url, data = {'query': query})
res = json.loads(response.text)

for item in res['results']['bindings']:
    print (item['dataset']['value'])
```

    http://wiki.linked.earth/Special:URIResolver/MD982181.Khider.2014
    http://wiki.linked.earth/Special:URIResolver/A7.Oppo.2005
    http://wiki.linked.earth/Special:URIResolver/MD982176.Stott.2004
    http://wiki.linked.earth/Special:URIResolver/BJ8-2D03-2D13GGC.Linsley.2010
    http://wiki.linked.earth/Special:URIResolver/BJ8-2D03-2D70GGC.Linsley.2010
    http://wiki.linked.earth/Special:URIResolver/MD01-2D2390.Steinke.2008
    http://wiki.linked.earth/Special:URIResolver/MD97-2D2141.Rosentha.2003
    http://wiki.linked.earth/Special:URIResolver/MD98-2D2165.Levi.2007
    http://wiki.linked.earth/Special:URIResolver/MD98-2D2170.Stott.2004
    http://wiki.linked.earth/Special:URIResolver/MD01-2D2378.Xu.2008


<div class="alert alert-warning" role="alert" style="margin: 10px">
<p>**NOTE**</p>
<p>We are working on a way to upload the LiPD files directly into the workspace</p>
</div>


```python
# Import needed packages
# Make the figure showing the location of the data
import lipd as lpd
import numpy as np
from scipy.stats.mstats import mquantiles
import matplotlib.pyplot as plt
import cartopy
import cartopy.crs as ccrs
import os as os
import pandas as pd
import matplotlib.colors as colors
import sys
import math
```

    
    Choose a loading option:
    1. Select specific file(s)
    2. Load entire folder
    Option: 2
    Where are your file(s) stored?
    1. Current
    2. Browse
    3. Downloads
    4. LiPD Workspace
    
    Option: 2


    Trying `CDLL(/usr/lib/libc.dylib)`
    Library path: '/usr/lib/libc.dylib'
    DLL: <CDLL '/usr/lib/libc.dylib', handle 7fff6761ad08 at 0x1281d8748>



```python
# Needed Functions to make the map
def map_all(dataframe,fignum, markersize = 150, saveFig = True):
    %matplotlib inline
    plt.figure(figsize=(6,6)) # intialize a figure (can change the size if necessary)
    geo_ax = plt.axes(projection=ccrs.Orthographic(120,0))
    l = geo_ax.stock_img() # Add topography
    l.set_alpha(.5) # Reduce transparency so I can see the markers
    geo_ax.add_feature(cartopy.feature.LAND,edgecolor = 'black', facecolor = 'none') # set the land borders
    geo_ax.set_global() #Set to a global perspective
    geo_ax.scatter(dataframe['Longitude'], dataframe['Latitude'],                  
           s = markersize,
           facecolors = '#047878',
           marker = 'o',
           lw = 2,
           transform = ccrs.Geodetic(),
            label = "Mg/Ca") #plot the data'
    #labels = ["Mg/Ca"]
    #Get the legend 
    plt.legend(scatterpoints=1, loc=3, fancybox=True, title='Proxy Observation', framealpha=0.8, shadow=True, fontsize=10)
    # Save the figure if needed
    if saveFig == True:
        plt.savefig(dir+"/Figures/Figure"+str(fignum)+".eps",\
        bbox_inches = 'tight', pad_inches = 0.25) 
        
# Set the working directory
dir = "/Users/deborahkhider/Documents/Documents/LinkedEarth/Holocene Project"

# Get the path for the LiPD files 
#path = lpd.path
path = "/Users/deborahkhider/Documents/Documents/LinkedEarth/Holocene Project/IPWP LiPD"

#load the LiPDs file
lpd.loadLipds() 

# Get the name of each record and store
files = os.listdir(path)

# Load the age and depth data files
# TODO: replace by writing the age model into each LiPD file
files2 = os.listdir(dir+"/IPWP Ages")
age_files = [i for i in files2 if "age" in i] 
depth_files = [i for i in files2 if "depth" in i] 
# Load the results of the spectral analysis for all the cores
spectral_files = os.listdir(dir+"/IPWP Spectral")
# Get a list of all the LiPD files in the directory
lipd_in_directory = [i for i in files if i.endswith('.lpd')] 

# Get the data for the 900-1200 year band
# Get the various data
index = 0 # initialize the stepping index

# Initialize empty lists and numpy arrays
record = []
lat = np.zeros(len(lipd_in_directory))
lon =np.zeros(len(lipd_in_directory))
med_per = np.zeros(len(lipd_in_directory))
HDR = np.zeros(len(lipd_in_directory))
percent_present = np.zeros(len(lipd_in_directory))
percent_var = np.zeros(len(lipd_in_directory))
mean_res = np.zeros(len(lipd_in_directory))
mean_ageU = np.zeros(len(lipd_in_directory))

for i in lipd_in_directory:
    # Get the metadata
    lipd_meta = lpd.getMetadata(i)
    lat[index] = lipd_meta['geo']['geometry']['coordinates'][0]
    lon[index] = lipd_meta['geo']['geometry']['coordinates'][1] 
    record.append(lipd_meta['dataSetName'])
    # open the age file and store the 95% quantile for age
    age = np.genfromtxt(dir+"/IPWP Ages/"+ \
        [j for j in age_files if record[index] in j][0], delimiter=',')
    ageQ = mquantiles(age, prob=[0.025, 0.975], axis=0)
    mean_ageU[index]=np.nanmean(ageQ[1:,]-ageQ[0:,])
    # Calculate the average resolution
    mean_res[index] = np.nanmean(np.diff(mquantiles(age, prob=[0.5], axis=0)))
    # open the file containing the information about the spectral analysis
    info = np.genfromtxt(dir+"/IPWP spectral/"+ \
        [k for k in spectral_files if record[index] in k][0], delimiter=',')
    med_per[index] = info[0,0]
    if np.isnan(info[0,1]) == 0: 
        HDR[index] = info[0,1]
    else:
        HDR[index]=300
    percent_present[index]= info[0,2]
    percent_var[index] = info[0,3]     
    index+=1                     

# Get everything into a dataframe
data_band1 = pd.DataFrame({"Record" : record,
                    "Latitude" : lat,
                    "Longitude" : lon,
                    "Median Periodicity (yr)": med_per,
                    "HDR (yr)": HDR,
                    "Percent present peaks": percent_present,
                    "Percent Variance": percent_var,
                    "Average Age Resolution": mean_res,
                    "Average Age Uncertainty": mean_ageU}) 

# Make the map (Don't save for the purpose of this notebook)
map_all(data_band1,1, markersize = 150, saveFig = False)
```

    Found: 10 LiPD file(s)
    processing: A7.Sun.2005.lpd
    processing: BJ8-13GGC.Linsley.2010.lpd
    processing: BJ8-70GGC.Linsley.2010.lpd
    processing: MD01-2390.Steinke.2008.lpd
    processing: MD012378.Xu.2008.lpd
    processing: MD97-2141.Rosenthal.2003.lpd
    processing: MD98-2165.Levi.2007.lpd
    processing: MD98-2170.Stott.2004.lpd
    processing: MD982176.Stott.2004.lpd
    processing: MD982181.Khider.2014.lpd
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete


    //anaconda/lib/python3.5/site-packages/matplotlib/artist.py:224: MatplotlibDeprecationWarning: get_axes has been deprecated in mpl 1.5, please use the
    axes property.  A removal date has not been set.
      stacklevel=1)



![png](output_4_2.png)


## Age model

The age models for each of the sedimentary records considered in this study were obtained using the Bchron software<sup>[5]</sup>. A Jupyter Notebook for each of the record is available upon request and contains information about reservoir age corrections and removal of outliers suggested by the original authors.

<div class="alert alert-warning" role="alert" style="margin: 10px">
<p>**NOTE**</p>
<p>Look at individual notebooks for age information!</p>
</div>

## Spectral Analysis
### Lomb-Scargle Periodogram
Holocene periodicities were inferred from the Lomb-Scargle periodogram<sup>[6]</sup>, which allows to handle unevenly-spaced timeseries, for each of the age model realizations (n=1000) returned by Bchron. We then identified the maximum peak within the 900-1200 year band, the 1200-2000 year band, and the 2000-3000 year band if present. The median periodicity is calculated at the median of the peak periodicities thus identified while the highest density region represents the 95% confidence interval on the peak periodicities.

The individual ensemble spectra for each of the sedimentary records are available in the individual Jupyter Notebooks. 

<div class="alert alert-warning" role="alert" style="margin: 10px">
<p>**NOTE**</p>
<p>We are currently working on adding an AR1 model!</p>
</div>

### Cross-Wavelet Analysis
We performed cross-wavelet analysis<sup>[7]</sup> between each realization of the sedimentary records and the TSI record of ref 9.

<div class="alert alert-warning" role="alert" style="margin: 10px">
<p>**NOTE**</p>
<p>TODO: Write the age model into the LiPD file and load them back into the workspace.<br>
In the meantime, use the csv file output from each of the Jupyter Notebook</p>
</div>


```python
# Get a Matlab session started
from pymatbridge import Matlab
mlab = Matlab()

mlab = Matlab(executable='/Applications/MATLAB_R2016a.app/bin/matlab')

mlab.start()
```

    Starting MATLAB on ZMQ socket ipc:///tmp/pymatbridge-4ca9fe0f-1eae-49fb-8e9a-2188d8b16241
    Send 'exit' command to kill the server
    ..........MATLAB started and connected!


    //anaconda/lib/python3.5/site-packages/IPython/nbformat.py:13: ShimWarning: The `IPython.nbformat` package has been deprecated. You should import from nbformat instead.
      "You should import from nbformat instead.", ShimWarning)





    <pymatbridge.pymatbridge.Matlab at 0x128c5e898>




```python
# Run the cross-wavelet analysis using Matlab
results = mlab.run_func('/Users/deborahkhider/Documents/MATLAB/AGU2016_XWavelet_Processing.m',{'x': 10}, nargout=4)
```


```python
# Pass the variables back into Python
median_angle1 = results['result'][0]
median_angle2 = results['result'][1]
HDR_angle1 = results['result'][2]
HDR_angle2 = results['result'][3]

# Create a dataframe
data = pd.DataFrame({"Lat": np.array([28,-5,-7.4,-3.57,6.63,8.78,-9.65,-10.6,-13.1,6.45]),
                    "Lon": np.array([127,133,115,119,113,121,118,125,122,126]),
                    "meantheta1": median_angle1[0],
                    "meantheta2": median_angle2[0],
                    "HDR1": HDR_angle1[0],
                    "HDR2": HDR_angle2[0]})
```


```python
# stop the Matlab session
mlab.stop()
```

    MATLAB closed





    True



## Results
### Holocene Periodicities within the Indo-Pacific Warm Pool
The median periodicity is expressed as the median peak of the ensemble spectra while the highest density region (HDR) represents the 95% confidence interval on the ensemble spectra.
#### 900-1200-yr band


```python
# Needed functions for the plot
def roundnearest(x,to):
    rounded = int(round(x/to))*to
    return rounded

def roundup(x,to):
    rounded = int(math.ceil(x/to))*to
    return rounded

def rounddown(x,to):
    rounded = int(math.floor(x/to))*to
    return rounded    
    
    
def plotSizeColor(dataframe,fignum, bin, rounding=10, markersize = 150, saveFig = True):  
    %matplotlib inline
    plt.figure(figsize=(6,6)) # intialize a figure (can change the size if necessary)
    #look for the index with the largest symbol (smallestt HDR)
    v = dataframe['HDR (yr)']
    index = v.idxmin()              
    # get the map ready
    geo_ax = plt.axes(projection=ccrs.Orthographic(120,0))
    l = geo_ax.stock_img() # Add topography
    l.set_alpha(.5) # Reduce transparency so I can see the markers
    geo_ax.add_feature(cartopy.feature.LAND,edgecolor = 'black', facecolor = 'none') # set the land borders
    geo_ax.set_global() #Set to a global perspective
    # Now make up data to be able to get a legend
    corr = markersize/(1/dataframe.mean()['HDR (yr)'])
    sizes = np.linspace(dataframe.min()['HDR (yr)'], dataframe.max()['HDR (yr)'], bin)
    step=np.diff(sizes)[0]
    labels=[str(rounddown(s,rounding))+'-'+str(rounddown(s+step,rounding))+' yrs' for s in sizes]
    points = [geo_ax.scatter(dataframe['Longitude'][index],dataframe['Latitude'][index],s=corr/s,facecolors='none', marker ='o', transform = ccrs.Geodetic()) for s in sizes]
    #plot the real data          
    cax = geo_ax.scatter(dataframe['Longitude'], dataframe['Latitude'],
           c = dataframe['Median Periodicity (yr)'],            
           s = corr/dataframe['HDR (yr)'],
           marker = 'o',
           lw = 2,
           cmap = 'Oranges',
           transform = ccrs.Geodetic()) #plot the data    
    #geo_ax1.set_title('Median Periodicity (yr)') # Set the title of the graph
    cbar = plt.colorbar(cax) # Add a colorbar
    cbar.set_label('Median Periodicity (yrs)') #Label the colorbar
    #Get the legend    
    plt.legend(points, labels, scatterpoints=1, loc=3, fancybox=True, title='Highest Density Region', framealpha=0.8, shadow=True, fontsize=10)
    # Save the figure if needed
    if saveFig == True:
        plt.savefig(dir+"/Figures/Figure"+str(fignum)+".eps",\
        bbox_inches = 'tight', pad_inches = 0.25)
        
# Since the data was already entered to make the database plot, run the plot function directly  
plotSizeColor(data_band1,6,3, markersize = 150, rounding=10, saveFig = False)
```

    //anaconda/lib/python3.5/site-packages/matplotlib/artist.py:224: MatplotlibDeprecationWarning: get_axes has been deprecated in mpl 1.5, please use the
    axes property.  A removal date has not been set.
      stacklevel=1)



![png](output_11_1.png)


Mean periodicity &#177; standard error = 1022 &#177; 27 years.

#### 1200-2000-yr band


```python
# Get the various data
index = 0 # initialize the stepping index

# Initialize empty lists and numpy arrays
record = []
lat = np.zeros(len(lipd_in_directory))
lon =np.zeros(len(lipd_in_directory))
med_per = np.zeros(len(lipd_in_directory))
HDR = np.zeros(len(lipd_in_directory))
percent_present = np.zeros(len(lipd_in_directory))
percent_var = np.zeros(len(lipd_in_directory))
mean_res = np.zeros(len(lipd_in_directory))
mean_ageU = np.zeros(len(lipd_in_directory))

for i in lipd_in_directory:
    # Get the metadata
    lipd_meta = lpd.getMetadata(i)
    lat[index] = lipd_meta['geo']['geometry']['coordinates'][0]
    lon[index] = lipd_meta['geo']['geometry']['coordinates'][1] 
    record.append(lipd_meta['dataSetName'])
    # open the age file and store the 95% quantile for age
    age = np.genfromtxt(dir+"/IPWP Ages/"+ \
        [j for j in age_files if record[index] in j][0], delimiter=',')
    ageQ = mquantiles(age, prob=[0.025, 0.975], axis=0)
    mean_ageU[index]=np.nanmean(ageQ[1:,]-ageQ[0:,])
    # Calculate the average resolution
    mean_res[index] = np.nanmean(np.diff(mquantiles(age, prob=[0.5], axis=0)))
    # open the file containing the information about the spectral analysis
    info = np.genfromtxt(dir+"/IPWP spectral/"+ \
        [k for k in spectral_files if record[index] in k][0], delimiter=',')
    med_per[index] = info[1,0]
    if np.isnan(info[1,1]) == 0: 
        HDR[index] = info[1,1]
    else:
        HDR[index]=800
    percent_present[index]= info[1,2]
    percent_var[index] = info[1,3]     
    index+=1                     

# Get everything into a dataframe
data_band2 = pd.DataFrame({"Record" : record,
                    "Latitude" : lat,
                    "Longitude" : lon,
                    "Median Periodicity (yr)": med_per,
                    "HDR (yr)": HDR,
                    "Percent present peaks": percent_present,
                    "Percent Variance": percent_var,
                    "Average Age Resolution": mean_res,
                    "Average Age Uncertainty": mean_ageU}) 
                    
#make the plot                    
plotSizeColor(data_band2,7,3, markersize = 150, rounding=10, saveFig = False)
```

    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete


    //anaconda/lib/python3.5/site-packages/matplotlib/artist.py:224: MatplotlibDeprecationWarning: get_axes has been deprecated in mpl 1.5, please use the
    axes property.  A removal date has not been set.
      stacklevel=1)



![png](output_14_2.png)


Mean periodicity &#177; standard error = 1486 &#177; 96 years.

#### 2000-3000 yr band


```python
# Get the various data
index = 0 # initialize the stepping index

# Initialize empty lists and numpy arrays
record = []
lat = np.zeros(len(lipd_in_directory))
lon =np.zeros(len(lipd_in_directory))
med_per = np.zeros(len(lipd_in_directory))
HDR = np.zeros(len(lipd_in_directory))
percent_present = np.zeros(len(lipd_in_directory))
percent_var = np.zeros(len(lipd_in_directory))
mean_res = np.zeros(len(lipd_in_directory))
mean_ageU = np.zeros(len(lipd_in_directory))

for i in lipd_in_directory:
    # Get the metadata
    lipd_meta = lpd.getMetadata(i)
    lat[index] = lipd_meta['geo']['geometry']['coordinates'][0]
    lon[index] = lipd_meta['geo']['geometry']['coordinates'][1] 
    record.append(lipd_meta['dataSetName'])
    # open the age file and store the 95% quantile for age
    age = np.genfromtxt(dir+"/IPWP Ages/"+ \
        [j for j in age_files if record[index] in j][0], delimiter=',')
    ageQ = mquantiles(age, prob=[0.025, 0.975], axis=0)
    mean_ageU[index]=np.nanmean(ageQ[1:,]-ageQ[0:,])
    # Calculate the average resolution
    mean_res[index] = np.nanmean(np.diff(mquantiles(age, prob=[0.5], axis=0)))
    # open the file containing the information about the spectral analysis
    info = np.genfromtxt(dir+"/IPWP spectral/"+ \
        [k for k in spectral_files if record[index] in k][0], delimiter=',')
    med_per[index] = info[2,0]
    if np.isnan(info[2,1]) == 0: 
        HDR[index] = info[2,1]
    else:
        HDR[index]=1000
    percent_present[index]= info[2,2]
    percent_var[index] = info[2,3]     
    index+=1                     

# Get everything into a dataframe
data_band3 = pd.DataFrame({"Record" : record,
                    "Latitude" : lat,
                    "Longitude" : lon,
                    "Median Periodicity (yr)": med_per,
                    "HDR (yr)": HDR,
                    "Percent present peaks": percent_present,
                    "Percent Variance": percent_var,
                    "Average Age Resolution": mean_res,
                    "Average Age Uncertainty": mean_ageU}) 
                    
#make the plot                    
plotSizeColor(data_band3,8,3, markersize = 150, rounding=10, saveFig = False)
```

    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete
    Process Complete


    //anaconda/lib/python3.5/site-packages/matplotlib/artist.py:224: MatplotlibDeprecationWarning: get_axes has been deprecated in mpl 1.5, please use the
    axes property.  A removal date has not been set.
      stacklevel=1)



![png](output_16_2.png)


Mean periodicity &#177; standard error = 2391 &#177; 138 years.

<p>Despite large uncertainties in the location of the spectral peak within each individual record arising from age model uncertainty, sea surface variability of ~1000 years, ~1500 years, and ~2500 years are present in at least 70-95% of the ensemble spectra.</p>

<p> Remarkably, all records suggest a periodicity near ~1500 years, reminiscent of the cycles characteristic of Marine Isotope Stage 3<sup>[8]</sup>. These cycles are absent from ecisting records of TSI<sup>[9]</sup>, questioning the millennial-scale solar-climate connection.

### The role of the sun


```python
# Make the figure based on the cross-wavelet analysis.
bin = 4 
rounding = 25
markersize =150

#1000-year period
%matplotlib inline
fig = plt.figure(figsize=(14,6)) # intialize a figure (can change the size if necessary)
#look for the index with the largest symbol (smallestt HDR)
v = data['HDR1']
index = v.idxmin()              
# get the map ready
geo_ax1 = plt.subplot(1,2,1, projection = ccrs.Orthographic(120,0))
l = geo_ax1.stock_img() # Add topography
l.set_alpha(.5) # Reduce transparency so I can see the markers
geo_ax1.add_feature(cartopy.feature.LAND,edgecolor = 'black', facecolor = 'none') # set the land borders
geo_ax1.set_global() #Set to a global perspective
# Now make up data to be able to get a legend
corr = markersize/(1/data.mean()['HDR1'])
sizes = np.linspace(data.min()['HDR1'], data.max()['HDR1'], bin)
step=np.diff(sizes)[0]
labels=[str(rounddown(s,rounding))+'-'+str(rounddown(s+step,rounding))+'$^\circ$' for s in sizes]
points = [geo_ax1.scatter(data['Lon'][index],data['Lat'][index],s=corr/s,facecolors='none', marker ='o', transform = ccrs.Geodetic()) for s in sizes]
#plot the real data 
bounds = np.linspace(-110, 110, 12)
norm = colors.BoundaryNorm(boundaries=bounds, ncolors=256)         
cax1 = geo_ax1.scatter(data['Lon'], data['Lat'],
       c = data['meantheta1'],            
       s = corr/data['HDR1'],
       marker = 'o',
       lw = 2,
       cmap = 'bwr',
       #vmin=-110,
       #vmax=110,
       norm=norm,
       transform = ccrs.Geodetic()) #plot the data 
#geo_ax1.set_title('Median Periodicity (yr)') # Set the title of the graph
#cbar = plt.colorbar(cax1) # Add a colorbar
#cbar.set_label('Median Phasing (deg)') #Label the colorbar
#Get the legend    
plt.legend(points, labels, scatterpoints=1, loc=3, fancybox=True, title='Highest Density Region', framealpha=0.8, shadow=True, fontsize=10)

# 2500-yr period

v = data['HDR2']
index = v.idxmin()              
# get the map ready
geo_ax2 = plt.subplot(1,2,2, projection = ccrs.Orthographic(120,0))
l = geo_ax2.stock_img() # Add topography
l.set_alpha(.5) # Reduce transparency so I can see the markers
geo_ax2.add_feature(cartopy.feature.LAND,edgecolor = 'black', facecolor = 'none') # set the land borders
geo_ax2.set_global() #Set to a global perspective
# Now make up data to be able to get a legend
corr = markersize/(1/data.mean()['HDR2'])
sizes = np.linspace(data.min()['HDR2'], data.max()['HDR2'], bin)
step=np.diff(sizes)[0]
labels=[str(rounddown(s,rounding))+'-'+str(rounddown(s+step,rounding))+'$^\circ$' for s in sizes]
points = [geo_ax2.scatter(data['Lon'][index],data['Lat'][index],s=corr/s,facecolors='none', marker ='o', transform = ccrs.Geodetic()) for s in sizes]
#plot the real data          
cax2 = geo_ax2.scatter(data['Lon'], data['Lat'],
       c = data['meantheta2'],            
       s = corr/data['HDR2'],
       marker = 'o',
       lw = 2,
       cmap = 'bwr',
       #vmin=-110,
       #vmax=110,
       norm=norm,
       transform = ccrs.Geodetic()) #plot the data 
#geo_ax1.set_title('Median Periodicity (yr)') # Set the title of the graph
#Get the legend    
plt.legend(points, labels, scatterpoints=1, loc=3, fancybox=True, title='Highest Density Region', framealpha=0.8, shadow=True, fontsize=10)
fig.subplots_adjust(right=0.8)
cbar_ax = fig.add_axes([0.85, 0.15, 0.02, 0.7])
#cbar = fig.colorbar(cax2) # Add a colorbar
cbar = fig.colorbar(cax2, cax=cbar_ax, ticks=bounds)
cbar.set_label('Median Phasing (deg)') #Label the colorbar

```

    //anaconda/lib/python3.5/site-packages/matplotlib/artist.py:224: MatplotlibDeprecationWarning: get_axes has been deprecated in mpl 1.5, please use the
    axes property.  A removal date has not been set.
      stacklevel=1)



![png](output_18_1.png)


<p>The mean phase &#177; standard deviation is -17&#177;55&#176; for the periodicity centered around ~1000 years and 10&#177;97&#176; for the periodicity centered around ~2500 years.</p>

## Future Work
<ol>
<li> <b>Expand the database</b> to global coverage. </li>
<li> <b>Explore lead-lag relationships</b> among various regions. </li>
<li> <b>Estimate magnitude</b> of millennial-scale variability. </li>
</ol>

## LinkedEarth and the future of paleoclimatology
<ul>
<li> <b>Crowdsource data curation</b> though a wiki platform. </li>
<li> <b> Develop web standards</b> for paleoclimate observations. </li>
<li> <b> Develop social codes</b> to accelerate scientific discovery. </li>
</ul>

<p style="font-weight:bold"> The ultimate goal of LinkedEarth is to allow scientists to spend more time on science, less on issues computers can solve. </p>

## Key points
<ol>
<li> The large age model uncertainty inherent to paleoceanographic reconstructions prevents meaningful analysis of periodicities from one single record. </li>
<li> Synthesis of multple records suggests robust periodicities centered around ~1000 years, ~1500 years and ~2500 years in sea surface temperature variability within the Indo-Pacific Warm Pool. </li>
</ol>
<p style="font-weight:bold"> We cannot discard the possibility that the sun might have paced millennial-scale climate variability over the Holocene. </p>
<ol>
<li> Refuting or accepting the solar forcing hypothesis will require additional records with a global coverage.</li>
<li> LinkedEarth aims to facilitate synthesis work to answer pressing questions in paleoclimatology. </li>
</ol>

## References
1.	Bond, G., et al., Persistent solar influence on North Atlantic climate during the Holocene. Science, 2001. 294(2130): p. 2130-2136.
2.	Debret, M., et al., The origin of the 1500-year climate cycles in Holocene North Atlantic record. Climate of the Past Discussions, 2007. 3: p. 679-692.
3.	Debret, M., et al., Evidence from wavelet analysis for a mid-Holocene transition in global climate forcing. Quaternary Science Reviews, 2009. 28(25-26): p. 2675-2688.
4.	Khider, D., C.S. Jackson, and L.D. Stott, Assessing millennial-scale variability during the Holocene: A perspective from the western tropical Pacific. Paleoceanography, 2014. 29(3): p. 143-159.
5.	Haslett, J. and A. Parnell, A simple monotone process with application to radiocarbon-dated depth chronologies. Journal of the Royal Statistical Society C, 2008. 57: p. 399-418.
6.	Lomb, N.R., Least-squares frequency analysis of unequally spaced data. Astrophysics and Space Science, 1976. 39: p. 447-462.
7.	Grinsted, A., J.C. Moore, and S. Jevrejeva, Application of the cross wavelet transform and wavelet coherence to geophysical time series. Nonlinear processes in Geophysics, 2004. 11: p. 561-566.
8.	Schulz, M., On the 1470-year pacing of Dansgaard-Oeschger warm events. Paleoceanography, 2002. 17(2).
9.	Steinhilber, F., et al., 9,400 years of cosmic radiation and solar activity from ice cores and tree rings. Proc Natl Acad Sci U S A, 2012. 109(16): p. 5967-71.

### Acknowledgements
This material is based upon work supported by the National Science Foundation under Grant Number ICER-1541029. Any opinions, findings, and conclusions or recommendations expressed in this material are those of the investigators and do not necessarily reflect the views of the National Science Foundation. 
