---
title: "IMDLIB"
date: 2020-01-21T03:32:14+05:30
description: A python package for handling IMD data.
draft: false
hideToc: false
enableToc: true
enableTocContent: true
author: Saswata Nandi
authorEmoji: 🤖
tags:
- hydrology
- IMD
- Softwares
- github
- pypi
- conda
categories:
- hydrology
- softwares
- pypi
featured_image: "feature1/imdlib1.png"
---

## Description

This is a python package to download and handle binary grided data from Indian Meterological department (IMD).

## Installation

IMDLIB can be installed via pip, conda, or using the source file from Github. It is tested for both Windows and Linux platforms with 64 bit architecture only.

Installation using pip.
``` bash
pip install imdlib
```

Installation using conda.
``` bash
conda install -c iamsaswata imdlib
```
Installation from source file.

``` bash
pip install git+https://github.com/iamsaswata/imdlib.git
```
IMDLIB is currently built with the following plugins. You need to have a python version >= 3.5 and install the below dependencies before installing IMDLIB from source.

| Plugin | Version No|
| ------ | ------ |
| certifi | [2019.11.28] |
| matplotlib | [3.1.3] |
| numpy | [1.18.1] |
| pandas | [0.25.3] |
| python-dateutil | [ 2.8.1 ] |
| pytest | [5.3.4] |
| pytz | [ 2019.3] |
| requests | [2.23.0] |
| scipy | [1.4.1] |
| six | [1.14.0] |
| xarray | [0.14.1] |

## Usage

### Downloading data from IMD

IMDLIB is capable of downloading gridded rainfall and temperature data (min and max) from [IMD](http://www.imdpune.gov.in/) Pune.

Special thanks to [Mr. Pratiman Patel](https://scholar.google.co.in/citations?user=u4ZxPmIAAAAJ&hl=en) for drafting this feature.


``` python
import imdlib as imd

# Downloading 8 years of rainfall data for India
start_yr = 2010
end_yr = 2018
variable = 'rain' # other options are ('tmin'/ 'tmax')
file_dir = 'dataFolder'
"""
fn_format   : str or None
        fn_format represent filename format. 
        Default vales is None.  Which means filesnames are accoding to the IMD naming convention
        If we specify fn_format = 'yearwise', filenames are renamed like <year.grd>

file_dir   : str or None
        Directory for downliading the files.
        If None, the currently working directory is used.

sub_dir : bool
		True : if you need subdirectory for each variable type;
        False: Files will be saved directly under main directory
proxies : dict
        Give details in curly bracket as shown in the example below
        e.g. proxies = { 'http' : 'http://uname:password@ip:port'}
"""
data = imd.get_data(variable, start_yr, end_yr, fn_format='yearwise', file_dir=file_dir)
```

### Reading downloaded binary dataset

One major purposes of IMDLIB is to process gridded IMD meterological dataset. The original data is available in .grd file format. IMDLIB can read .grd file in xarray style and will create a *IMD class* objetct. For the already downloaded data, one should use the <code> imd.open_data()</code> functionality for further analysis and processing.


``` python
import imdlib as imd

start_yr =2018
end_yr = 2018
variable = 'rain' # other options are ('tmin'/ 'tmax')
file_format = 'yearwise' # other option (None), which will assume deafult imd naming convention
file_dir = '/home/data/imd' # other option keep it blank
data = imd.open_data(variable, start_yr, end_yr,'yearwise', file_dir)
data
```
    <imdlib.core.IMD at 0x7f39b0960750>

> *file_dir* should refer to *top-lev* directory. It should contain 3 sub-directories. <mark>*rain*</mark>, <mark>*tmin*</mark>, and <mark>*tmax*</mark>.

> if *file_dir* exist, but no subdirectory, it will try to find the files in *file_dir*. But be careful if you are using `file_format = 'yearwise'`, as it will not differentiate between *2018.grd* for rainfall and *2018.grd* for tmin.

> if *file_dir* is not given, it will look for the associate subdirectories and files from the current directory.

### Get shape of IMD objecct
```python
data.shape
```
    (365, 135 129)

### Get numpy. ndarray

```python
np_array = data.data
```


### Get xarray object


```python
xr_objecct = data.get_xarray()
type(xr_objecct)
```
    xarray.core.dataarray.DataArray

### Display mean rainfall in Map

```python
xr_objecct.mean('time').plot()
```
![png](/softwares/imdlib1.png)


### Processing and Saving

```python
# Get data for a given coordinare and convert to csv file
lat = 20.03
lon = 77.23
out_dir='/home/downloads/data'

data.to_csv(file_name, lat, lon, out_dir)

# Convert to netcdf file
data.to_netcdf(file_name, out_dir)

# Convert to GeoTIFF format
data.to_geotiff(file_name, out_dir)

```

<div style="text-align: justify"> For converting to GeoTIFF, we are using [`rioxarray`](https://corteva.github.io/rioxarray/stable/) package, but it has not been considered as a dependency for IMDLIB. So, if a call is made to `to_geotiff()` functionaality, IMDLIB will check if *rioxarray* module is available to the system python path and if *rioxarray* is found, the corresponding GeoTIFF file will be generated or else a error will be thrown saying *rioxarray* is not installed. </div>


### NetCDF File Convention

<div style="text-align: justify"> The IMDLIB focuses on producing netCDF (network Common Data Form) based final output as netCDF is the format most commonly used for climate model generated data. The netCDF Climate and Forecast (CF) Metadata Conventions, Version 1.7, has been adopted by IMDLIB for its efficient and consistent use with other standard netCDF based tool/applications. The <span style="background-color:AliceBlue;color:LightSeaGreen">epsg:4326</span> coordinate reference systems (CRS) is considered in CF naming convention and is vital for the function <span style="background-color:AliceBlue;color:LightSeaGreen">to_geotiff</span> to work correctly. For more information on the CF convention, users are requested to visit <span><a href="https://cfconventions.org">CF Conventions Home Page </a></span> and <span><a href="https://unidata.github.io/python-training/workshop/XArray/xarray-and-cf">XArray & CF integration </a></span> resources.</div>

```shell
test@test:~/data$ ncdump -h test.nc 
netcdf test {
dimensions:
	time = 365 ;
	lat = 31 ;
	lon = 31 ;
variables:
	double tmax(time, lat, lon) ;
		tmax:_FillValue = NaN ;
		tmax:units = "C" ;
		tmax:long_name = "Maximum Temperature" ;
	double lat(lat) ;
		lat:_FillValue = NaN ;
		lat:axis = "Y" ;
		lat:standard_name = "latitude" ;
		lat:long_name = "latitude" ;
		lat:units = "degrees_north" ;
	double lon(lon) ;
		lon:_FillValue = NaN ;
		lon:axis = "X" ;
		lon:long_name = "longitude" ;
		lon:units = "degrees_east" ;
	int64 time(time) ;
		time:standard_name = "time" ;
		time:long_name = "time" ;
		time:units = "days since 2010-01-01" ;
		time:calendar = "proleptic_gregorian" ;

// global attributes:
		:Conventions = "CF-1.7" ;
		:title = "IMD gridded data" ;
		:source = "https://imdpune.gov.in/" ;
		:history = "2020-12-29 22:16:07.359709 Python" ;
		:references = "" ;
		:comment = "" ;
		:crs = "epsg:4326" ;
}
```


### License

IMDLIB is available under [MIT](https://opensource.org/licenses/MIT) the open source license.


