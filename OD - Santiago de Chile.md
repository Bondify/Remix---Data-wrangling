
The files used in this notebook can be found in this folder:
https://www.dropbox.com/home/Remix/Growth/International/Agencies%20by%20country/Chile/DPTM%20(Santiago%2C%20Chile)/OD/raw%20files%20-%20don't%20use%20these

For the shapefile we used the file called `zones_shapefile.shp`
For the OD data we used the file called `Abril2017.MatrizOD.csv`


```python
# Import libraries

import pandas as pd
import geopandas as gpd
import numpy as np
```


```python
# Ingest the data
# Change the path to your computer's

shapefile_path = "/Users/santiagotoso/GoogleDrive/Master/Python/Santiago de Chile/MatrizOD_Subidas_Bajadas_2017.04/zones_shapefile/zones_shapefile.shp" 
od_input_path = "/Users/santiagotoso/Downloads/MatrizOD_Subidas_Bajadas_2017.04/Abril2017.MatrizOD.csv"
shapefile = gpd.read_file(shapefile_path)
od = pd.read_csv(od_input_path, sep=';')
```

# Data files


```python
od.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Periodo</th>
      <th>ZonaSubida</th>
      <th>ZonaBajada</th>
      <th>nViajesSinBajada</th>
      <th>nViajesConBajada</th>
      <th>Expansion</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01 - PRE NOCTURNO</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01 - PRE NOCTURNO</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01 - PRE NOCTURNO</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01 - PRE NOCTURNO</td>
      <td>0</td>
      <td>100</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01 - PRE NOCTURNO</td>
      <td>0</td>
      <td>101</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check that we have a square metrix

print("Number of origin zones in od = " + str(len(od.ZonaSubida.unique())))
print("Number of detination zones = " + str(len(od.ZonaBajada.unique())))
```

    Number of origin zones in od = 803
    Number of detination zones = 803



```python
# Check different time periods

print(od.Periodo.unique())
```

    ['01 - PRE NOCTURNO' '02 - NOCTURNO' '03 - TRANSICION NOCTURNO'
     '04 - PUNTA MANANA' '05 - TRANSICION PUNTA MANANA'
     '06 - FUERA DE PUNTA MANANA' '07 - PUNTA MEDIODIA'
     '08 - FUERA DE PUNTA TARDE' '09 - PUNTA TARDE'
     '10 - TRANSICION PUNTA TARDE' '11 - FUERA DE PUNTA NOCTURNO'
     '12 - PRE NOCTURNO']



```python
# Rename the columns
# Filter only the time period the client wants to use

od = od.rename(index=str, columns={"ZonaSubida": "origin_id", "ZonaBajada": "destination_id", "Expansion": "count"})

manana = '04 - PUNTA MANANA'
od_manana = od[od.Periodo == manana][["origin_id","destination_id", 'count']]
od_manana.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>origin_id</th>
      <th>destination_id</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1934427</th>
      <td>0</td>
      <td>0</td>
      <td>7,3854599</td>
    </tr>
    <tr>
      <th>1934428</th>
      <td>0</td>
      <td>1</td>
      <td>4,923639774</td>
    </tr>
    <tr>
      <th>1934429</th>
      <td>0</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1934430</th>
      <td>0</td>
      <td>100</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1934431</th>
      <td>0</td>
      <td>101</td>
      <td>3,69272995</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Change "," for "." in the numbers as string

def point_for_comma(str):
    return float(str.replace(",", "."))

od_manana['count'] = od_manana['count'].apply(point_for_comma)
od_manana.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>origin_id</th>
      <th>destination_id</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1934427</th>
      <td>0</td>
      <td>0</td>
      <td>7.38546</td>
    </tr>
    <tr>
      <th>1934428</th>
      <td>0</td>
      <td>1</td>
      <td>4.92364</td>
    </tr>
    <tr>
      <th>1934429</th>
      <td>0</td>
      <td>10</td>
      <td>0.00000</td>
    </tr>
    <tr>
      <th>1934430</th>
      <td>0</td>
      <td>100</td>
      <td>0.00000</td>
    </tr>
    <tr>
      <th>1934431</th>
      <td>0</td>
      <td>101</td>
      <td>3.69273</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save the data file

od_manana.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Santiago de Chile/MatrizOD_Subidas_Bajadas_2017.04/od_santiago.csv", index = False)
```

# Shapefile


```python
# Get the ID from the shapefile and create the id field 
# Notice that the zone ID is inside the "descriptio" filed 

import re

def id_extractor(str):
    x = re.split('>', str)[5]
    y = x[:-4] 
    return y

# Create the column "area_id" with the zone ID we just extracted

shapefile['area_id']  = shapefile.descriptio.apply(id_extractor)
shapefile.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>descriptio</th>
      <th>timestamp</th>
      <th>begin</th>
      <th>end</th>
      <th>altitudeMo</th>
      <th>tessellate</th>
      <th>extrude</th>
      <th>visibility</th>
      <th>drawOrder</th>
      <th>icon</th>
      <th>geometry</th>
      <th>area_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>584</td>
      <td>&lt;table&gt;&lt;tr&gt;&lt;td&gt;Zona777&lt;/td&gt;&lt;td&gt;625&lt;/td&gt;&lt;/tr&gt;&lt;t...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1</td>
      <td>0</td>
      <td>-1</td>
      <td>None</td>
      <td>None</td>
      <td>POLYGON Z ((-70.543571 -33.548025 0, -70.54293...</td>
      <td>625</td>
    </tr>
    <tr>
      <th>1</th>
      <td>770</td>
      <td>&lt;table&gt;&lt;tr&gt;&lt;td&gt;Zona777&lt;/td&gt;&lt;td&gt;806&lt;/td&gt;&lt;/tr&gt;&lt;t...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1</td>
      <td>0</td>
      <td>-1</td>
      <td>None</td>
      <td>None</td>
      <td>POLYGON Z ((-70.787311 -33.564826 0, -70.79797...</td>
      <td>806</td>
    </tr>
    <tr>
      <th>2</th>
      <td>596</td>
      <td>&lt;table&gt;&lt;tr&gt;&lt;td&gt;Zona777&lt;/td&gt;&lt;td&gt;634&lt;/td&gt;&lt;/tr&gt;&lt;t...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1</td>
      <td>0</td>
      <td>-1</td>
      <td>None</td>
      <td>None</td>
      <td>POLYGON Z ((-70.797974 -33.555577 0, -70.80288...</td>
      <td>634</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Give the shapefile the correct projection
# Create a new geo data frame with the relevant variables

shapefile.crs = {'init': 'epsg:4326'}
shapefile_output = shapefile[['area_id', 'Name', 'geometry']]
shapefile_output.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>area_id</th>
      <th>Name</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>625</td>
      <td>584</td>
      <td>POLYGON Z ((-70.543571 -33.548025 0, -70.54293...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>806</td>
      <td>770</td>
      <td>POLYGON Z ((-70.787311 -33.564826 0, -70.79797...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>634</td>
      <td>596</td>
      <td>POLYGON Z ((-70.797974 -33.555577 0, -70.80288...</td>
    </tr>
  </tbody>
</table>
</div>




```python
import fiona; fiona.supported_drivers
```




    {'AeronavFAA': 'r',
     'ARCGEN': 'r',
     'BNA': 'raw',
     'DXF': 'raw',
     'CSV': 'raw',
     'OpenFileGDB': 'r',
     'ESRI Shapefile': 'raw',
     'GeoJSON': 'rw',
     'GPKG': 'rw',
     'GML': 'raw',
     'GPX': 'raw',
     'GPSTrackMaker': 'raw',
     'Idrisi': 'r',
     'MapInfo File': 'raw',
     'DGN': 'raw',
     'S57': 'r',
     'SEGY': 'r',
     'SUA': 'r'}




```python
# Save the geo data frame as a shapefile

shapefile_output.to_file(driver = 'ESRI Shapefile',
                         #crs_wkt = prj,
                         filename = "/Users/santiagotoso/GoogleDrive/Master/Python/Santiago de Chile/MatrizOD_Subidas_Bajadas_2017.04/output_shapefile/output_shapefile.shp" )
```

    /Users/santiagotoso/anaconda3/lib/python3.6/site-packages/geopandas/io/file.py:108: FionaDeprecationWarning: Use fiona.Env() instead.
      with fiona.drivers():



```python
# Does it look good?

test = gpd.read_file("/Users/santiagotoso/GoogleDrive/Master/Python/Santiago de Chile/MatrizOD_Subidas_Bajadas_2017.04/output_shapefile/output_shapefile.shp" )
test.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>area_id</th>
      <th>Name</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>625</td>
      <td>584</td>
      <td>POLYGON Z ((-70.543571 -33.548025 0, -70.54293...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>806</td>
      <td>770</td>
      <td>POLYGON Z ((-70.787311 -33.564826 0, -70.79797...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>634</td>
      <td>596</td>
      <td>POLYGON Z ((-70.797974 -33.555577 0, -70.80288...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>487</td>
      <td>457</td>
      <td>POLYGON Z ((-70.566011 -33.550304 0, -70.56159...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>439</td>
      <td>384</td>
      <td>POLYGON Z ((-70.73079 -33.594186 0, -70.735468...</td>
    </tr>
  </tbody>
</table>
</div>


