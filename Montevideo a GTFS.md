
# Montevideo data tranformation to GTFS

First of all we should the data from their website.

The file we'll use to create stop_times.txt can be found here: http://www.montevideo.gub.uy/sites/default/files/datos/uptu_pasada_variante.zip

The file we'll use to create stops.txt can be found here: http://intgis.montevideo.gub.uy/sit/php/common/datos/generar_zip2.php?nom_tab=v_uptu_paradas&tipo=gis

The file we'll use to create routes.txt can be found here: http://intgis.montevideo.gub.uy/sit/php/common/datos/generar_zip2.php?nom_tab=v_uptu_lsv_destinos&tipo=gis


```python
# Import the libraries needed

import pandas as pd
import geopandas as gpd
import numpy as np
```


```python
# Replace the paths for the path to the files in your computer

url_stop_times = "/Users/santiagotoso/GoogleDrive/Master/R/Remix/Montevideo/uptu_pasada_variante.csv"
s_path = "/Users/santiagotoso/GoogleDrive/Master/R/Remix/Montevideo/stops/v_uptu_paradas.shp"
rt_path = "/Users/santiagotoso/GoogleDrive/Master/R/Remix/Montevideo/v_uptu_lsv_destinos/v_uptu_lsv_destinos.shp"
```


```python
# Import the data as data frames and geo data frames

st = pd.read_csv(url_stop_times, sep=';')
s = gpd.read_file(s_path)
rt = gpd.read_file(rt_path)
```


```python
# We check the projection of the file. 
# If it doesn't have any you can always drag and drop the file to QGIS 
# go to "Properties" of the layer and get the projection from there.

s.crs
rt.crs
```




    {}




```python
# In this case it didn't have projection information so we
# looked for it in QGIS
# We set the right projection before changing it to the one we need

s.crs = {'init': 'epsg:32721'}
rt.crs = {'init': 'epsg:32721'}
```


```python
# We reproject to the system we need to create the GTFS

strans = s.to_crs({'init': 'epsg:4326'})
rtrans = rt.to_crs({'init': 'epsg:4326'})

# We check our projection is right now

strans.crs
```




    {'init': 'epsg:4326'}



## Agency.txt


```python
# Create the data frame

d = {'agency_id': [1], 
     'agency_name': ["STM Sistema de Transporte Metropolitano"], 
     'agency_url':["http://www.montevideo.gub.uy/transito-y-transporte/stm-sistema-de-transporte-metropolitano"],
     'agency_timezone':["America/Montevideo"]}
agency = pd.DataFrame(data = d, index = [''])
agency
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
      <th>agency_id</th>
      <th>agency_name</th>
      <th>agency_url</th>
      <th>agency_timezone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th></th>
      <td>1</td>
      <td>STM Sistema de Transporte Metropolitano</td>
      <td>http://www.montevideo.gub.uy/transito-y-transp...</td>
      <td>America/Montevideo</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save the data frame
# Change the path to your output desired address

agency.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/agency.txt", index = False)
```

## Calendar.txt


```python
# Create the data frame

d = {'service_id': [1,2,3], 
     'monday':[1,0,0],
     'tuesday': [1,0,0],
     'wednesday':[1,0,0],
     'thursday':[1,0,0],
     'friday':[1,0,0],
     'saturday':[0,1,0],
     'sunday':[0,0,1],
     'start_date':['20190101', '20190101', '20190101'],
     'end_date':['20191231', '20191231', '20191231']
    }

calendar = pd.DataFrame(data = d)
calendar
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
      <th>service_id</th>
      <th>monday</th>
      <th>tuesday</th>
      <th>wednesday</th>
      <th>thursday</th>
      <th>friday</th>
      <th>saturday</th>
      <th>sunday</th>
      <th>start_date</th>
      <th>end_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>20190101</td>
      <td>20191231</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>20190101</td>
      <td>20191231</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>20190101</td>
      <td>20191231</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save the data frame
# Change the path to your output desired address

calendar.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/calendar.txt", index = False)
```

## Stops.txt


```python
# First we search for NA values and save them adding a '-'
# This will helps us when we create the data frame

boolean_array = strans['ESQUINA'].isna()
strans.loc[boolean_array, ['ESQUINA']] = ' '
strans[['CALLE','ESQUINA']].isna().any()
```




    CALLE      False
    ESQUINA    False
    dtype: bool




```python
# Create the data frame

stops = pd.DataFrame(columns = ['stop_id', 'stop_name', 'stop_lat', 'stop_lon'])
stops['stop_id'] = strans["COD_UBIC_P"].astype('float').astype('int').astype('str').unique()
stops['stop_name'] = 'calle ' + strans.CALLE + " - esquina " + strans.ESQUINA + ' - id ' + stops.stop_id
stops['stop_name'] = stops.stop_name.unique()
stops['stop_lat'] = strans.geometry.y.unique()
stops['stop_lon'] = strans.geometry.x.unique()
len(stops)
```




    4712




```python
# Save the data frame
# Change the path to your output desired address

stops.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/stops.txt", index = False)
```

# Routes.txt


```python
# Create the fields we need for the GTFS from their data

route_id = rtrans.COD_SUBLIN.astype('str') + '-' + rtrans.DESC_SUBLI.astype('str')
route_id = route_id.unique()
route_short_name = rtrans.COD_LINEA.astype('str') + '-' + rtrans.COD_SUBLIN.astype('str') + '-' + rtrans.DESC_SUBLI.astype('str')
route_short_name = route_short_name.unique()
route_short_name = pd.DataFrame(data = route_short_name)
```


```python
# Create the data frame

routes = pd.DataFrame(data = route_id)
routes.columns = ['route_id']
routes['agency_id'] = '1'
routes['route_short_name'] = route_short_name
routes['route_long_name'] = ''
routes['route_type'] = '3'
routes.head()
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
      <th>route_id</th>
      <th>agency_id</th>
      <th>route_short_name</th>
      <th>route_long_name</th>
      <th>route_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1-CIUDAD VIEJA - MALVIN</td>
      <td>1</td>
      <td>1-1-CIUDAD VIEJA - MALVIN</td>
      <td></td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2-CJO. J. DE AMÃRICA - PCIO. DE LA LUZ</td>
      <td>1</td>
      <td>2-2-CJO. J. DE AMÃRICA - PCIO. DE LA LUZ</td>
      <td></td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3-PEÃAROL - PARQUE RODÃ</td>
      <td>1</td>
      <td>3-3-PEÃAROL - PARQUE RODÃ</td>
      <td></td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>246-GRUTA DE LOURDES - PARQUE RODO</td>
      <td>1</td>
      <td>3-246-GRUTA DE LOURDES - PARQUE RODO</td>
      <td></td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4-PORTONES - PLAZA ESPAÃA</td>
      <td>1</td>
      <td>4-4-PORTONES - PLAZA ESPAÃA</td>
      <td></td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save the data frame
# Change the path to your output desired address

routes.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/routes.txt", index = False)
```

# Trips.txt

In st some rows are from the same trip but have different `tipo_dia`.
This is because `tipo_dia` changes when you go beyond 24:00h.
When this happens the column `dia_anterior` has the value `S`.
In this case this breaks our pattern for calculating trip_id and we'll
need to change tipo_dia for those rows with `dia_anterior = S`.
Another problem is that `dia_anterior = *` for trips that take place after a Sunday at 24:00, but gets the `tipo_dia = 3` anyway.
This makes our formula for the `trip_id` not good cause two trips could
have the same `id` even they are not the same.
Looks like the first thing to do is to correct the times (`hora` and `frecuencia`)
and `tipo_dia`.


```python
# We start by joining the data frame with the stop times and the data frame with the routes
# Take into account that we'll consider the each combination "COD_SUBLIN"-"DESC_SUBLI" as 
# a route_id and each pattern inside of those will be the patterns.
# The only unique fields are the one indicating the patterns: "cod_variante" and "COD_VARIAN"
# respectively.

rtrans['route_id'] = rtrans.COD_SUBLIN.astype('str') + '-' + rtrans.DESC_SUBLI.astype('str')
st1 = pd.merge(st, rtrans, how='left', left_on=['cod_variante'], right_on=['COD_VARIAN'])
st1.head()
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
      <th>tipo_dia</th>
      <th>cod_variante</th>
      <th>frecuencia</th>
      <th>cod_ubic_parada</th>
      <th>ordinal</th>
      <th>hora</th>
      <th>dia_anterior</th>
      <th>GID</th>
      <th>COD_LINEA</th>
      <th>DESC_LINEA</th>
      <th>...</th>
      <th>DESC_SUBLI</th>
      <th>COD_VARIAN</th>
      <th>DESC_VARIA</th>
      <th>COD_VAR_01</th>
      <th>COD_ORIGEN</th>
      <th>DESC_ORIGE</th>
      <th>COD_DESTIN</th>
      <th>DESC_DESTI</th>
      <th>geometry</th>
      <th>route_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2078</td>
      <td>1</td>
      <td>2253</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>PORTONES - NUEVO CENTRO</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2079</td>
      <td>2</td>
      <td>2253</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>PORTONES - NUEVO CENTRO</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2340</td>
      <td>3</td>
      <td>2255</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>PORTONES - NUEVO CENTRO</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>7973</td>
      <td>22480</td>
      <td>2078</td>
      <td>1</td>
      <td>2249</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>PORTONES - NUEVO CENTRO</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>7973</td>
      <td>22480</td>
      <td>2079</td>
      <td>2</td>
      <td>2249</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>PORTONES - NUEVO CENTRO</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>




```python
# First we have to create the column "arrival_time" with the format needed for GTFS.
# This will allow us to solve the problem with "dia_anterio" and "tipo_dia" described
# previously.
# We start by creating the function that'll do this

def arrival_time(hora):
    'Creates the arrival_time field following the GTFS standard.'
    if len(str(hora)) == 1:
        return '00:0' + str(hora) + ':00'
    elif len(str(hora)) == 2:
        return '00:' + str(hora) + ':00'
    elif len(str(hora)) == 3:
        return '0' + str(hora)[0] + ':' + str(hora)[-2:] + ':00'
    elif len(str(hora)) == 4:
        return str(hora)[:-2] + ':' + str(hora)[-2:] +  ':00'
    else:
        return 'Not Known'
```


```python
# Now we can create the column

st1['arrival_time'] = st1['hora'].apply(arrival_time)
st1.head()
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
      <th>tipo_dia</th>
      <th>cod_variante</th>
      <th>frecuencia</th>
      <th>cod_ubic_parada</th>
      <th>ordinal</th>
      <th>hora</th>
      <th>dia_anterior</th>
      <th>GID</th>
      <th>COD_LINEA</th>
      <th>DESC_LINEA</th>
      <th>...</th>
      <th>COD_VARIAN</th>
      <th>DESC_VARIA</th>
      <th>COD_VAR_01</th>
      <th>COD_ORIGEN</th>
      <th>DESC_ORIGE</th>
      <th>COD_DESTIN</th>
      <th>DESC_DESTI</th>
      <th>geometry</th>
      <th>route_id</th>
      <th>arrival_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2078</td>
      <td>1</td>
      <td>2253</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:53:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2079</td>
      <td>2</td>
      <td>2253</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:53:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2340</td>
      <td>3</td>
      <td>2255</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:55:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>7973</td>
      <td>22480</td>
      <td>2078</td>
      <td>1</td>
      <td>2249</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:49:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>7973</td>
      <td>22480</td>
      <td>2079</td>
      <td>2</td>
      <td>2249</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>7973.0</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:49:00</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>




```python
# Correct the day type

def tipo_dia(dia_anterior, tipo):
    if dia_anterior != 'S':
        return tipo
    else:
        return tipo - 1

st1['tipo_dia'] = np.vectorize(tipo_dia)(st1['dia_anterior'], st1['tipo_dia'])
```


```python
# Create the trip_id

st1['trip_id'] =  st1['tipo_dia'].map(str) + '-' + st1['cod_variante'].map(str) + '-' + st1['frecuencia'].map(str)
st1.head()
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
      <th>tipo_dia</th>
      <th>cod_variante</th>
      <th>frecuencia</th>
      <th>cod_ubic_parada</th>
      <th>ordinal</th>
      <th>hora</th>
      <th>dia_anterior</th>
      <th>GID</th>
      <th>COD_LINEA</th>
      <th>DESC_LINEA</th>
      <th>...</th>
      <th>DESC_VARIA</th>
      <th>COD_VAR_01</th>
      <th>COD_ORIGEN</th>
      <th>DESC_ORIGE</th>
      <th>COD_DESTIN</th>
      <th>DESC_DESTI</th>
      <th>geometry</th>
      <th>route_id</th>
      <th>arrival_time</th>
      <th>trip_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2078</td>
      <td>1</td>
      <td>2253</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:53:00</td>
      <td>1-7973-22520</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2079</td>
      <td>2</td>
      <td>2253</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:53:00</td>
      <td>1-7973-22520</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>7973</td>
      <td>22520</td>
      <td>2340</td>
      <td>3</td>
      <td>2255</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:55:00</td>
      <td>1-7973-22520</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>7973</td>
      <td>22480</td>
      <td>2078</td>
      <td>1</td>
      <td>2249</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:49:00</td>
      <td>2-7973-22480</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>7973</td>
      <td>22480</td>
      <td>2079</td>
      <td>2</td>
      <td>2249</td>
      <td>N</td>
      <td>46722127.0</td>
      <td>81.0</td>
      <td>173</td>
      <td>...</td>
      <td>B</td>
      <td>7966.0</td>
      <td>138.0</td>
      <td>NUEVO CENTRO</td>
      <td>58.0</td>
      <td>LUIS A DE HERRERA Y GRAL FLORES</td>
      <td>LINESTRING (-56.16889034436939 -34.86981536802...</td>
      <td>972-PORTONES - NUEVO CENTRO</td>
      <td>22:49:00</td>
      <td>2-7973-22480</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 24 columns</p>
</div>




```python
# Drop duplicate rows (trips)

st1 = st1.sort_values(['tipo_dia', 'cod_variante', 'frecuencia', 'ordinal'])
st1 = st1.drop_duplicates(subset=['tipo_dia', 'cod_variante', 'frecuencia', 'cod_ubic_parada', 'ordinal', 'hora',
                                  'GID', 'route_id', 'arrival_time', 'trip_id'], keep=False)
```


```python
# Drop the trips that don't have a route_id
# We find that a small number of trips didn't have any match when joining
# with "routes". We can drop these without a big loss of data.

trips = st1[st1.route_id.notnull()]
```


```python
# Group the information by trip 

trips = trips.groupby(['route_id', 'tipo_dia', 'trip_id', 'GID'], as_index = False).agg({'ordinal': 'min'})
trips = trips.iloc[:, 0:4]
trips.columns = ('route_id', 'service_id', 'trip_id', 'shape_id')
trips.head()
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
      <th>route_id</th>
      <th>service_id</th>
      <th>trip_id</th>
      <th>shape_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1-CIUDAD VIEJA - MALVIN</td>
      <td>1</td>
      <td>1-1-10090</td>
      <td>19462120.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1-CIUDAD VIEJA - MALVIN</td>
      <td>1</td>
      <td>1-1-10270</td>
      <td>19462120.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1-CIUDAD VIEJA - MALVIN</td>
      <td>1</td>
      <td>1-1-10440</td>
      <td>19462120.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1-CIUDAD VIEJA - MALVIN</td>
      <td>1</td>
      <td>1-1-11000</td>
      <td>19462120.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1-CIUDAD VIEJA - MALVIN</td>
      <td>1</td>
      <td>1-1-11160</td>
      <td>19462120.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save the data frame
# Change the path to your output desired address

trips.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/trips.txt", index = False)
```

# Stop_times.txt


```python
# Get the columns we care about
# Change the name of some of them
# Create the columns we're missing
# Filter only the values that have a trip_id that is in trips.txt

stop_times = st1[["trip_id","ordinal", "cod_ubic_parada", "arrival_time"]]
stop_times = stop_times.rename(index=str, columns={"cod_ubic_parada": 'stop_id', "ordinal": "stop_sequence"})
stop_times['departure_time'] = stop_times['arrival_time']
stop_times = stop_times[stop_times.trip_id.isin(trips.trip_id)]
stop_times.head()
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
      <th>trip_id</th>
      <th>stop_sequence</th>
      <th>stop_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2205877</th>
      <td>1-1-5300</td>
      <td>1</td>
      <td>4041</td>
      <td>05:30:00</td>
      <td>05:30:00</td>
    </tr>
    <tr>
      <th>2205878</th>
      <td>1-1-5300</td>
      <td>2</td>
      <td>4763</td>
      <td>05:31:00</td>
      <td>05:31:00</td>
    </tr>
    <tr>
      <th>2205879</th>
      <td>1-1-5300</td>
      <td>3</td>
      <td>4764</td>
      <td>05:32:00</td>
      <td>05:32:00</td>
    </tr>
    <tr>
      <th>2205880</th>
      <td>1-1-5300</td>
      <td>4</td>
      <td>4765</td>
      <td>05:33:00</td>
      <td>05:33:00</td>
    </tr>
    <tr>
      <th>2205881</th>
      <td>1-1-5300</td>
      <td>5</td>
      <td>4773</td>
      <td>05:33:00</td>
      <td>05:33:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check if the stop_id has the same format in stop_times and stops

print(type(stop_times.stop_id[1]))
print(type(stops.stop_id[1]))
```

    <class 'numpy.int64'>
    <class 'str'>



```python
# Drop for duplicated trips in stop_times

stop_times = stop_times.drop_duplicates(subset=['trip_id', 'stop_sequence'], keep=False)
```


```python
# Match the data type for stop_id
# Merge stop_times and stops to check if there are any stops 
# that don't exist in stops.txt
stop_times['stop_id'] = stop_times.stop_id.astype(str)
stop_times = pd.merge(stop_times, stops, how='left', left_on=['stop_id'], right_on=['stop_id'])
stop_times.isnull().values.any()
```




    True




```python
# Drop the stops that are not in stops.txt

stop_times = stop_times.dropna(subset=['stop_name'])
stop_times.isnull().values.any()
```




    False




```python
# Keep only the columns that are interesting for us

stop_times = stop_times[['trip_id', 'stop_sequence', 'stop_id', 'arrival_time', 'departure_time']]
stop_times.head()
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
      <th>trip_id</th>
      <th>stop_sequence</th>
      <th>stop_id</th>
      <th>arrival_time</th>
      <th>departure_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1-1-5300</td>
      <td>1</td>
      <td>4041</td>
      <td>05:30:00</td>
      <td>05:30:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1-1-5300</td>
      <td>2</td>
      <td>4763</td>
      <td>05:31:00</td>
      <td>05:31:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1-1-5300</td>
      <td>3</td>
      <td>4764</td>
      <td>05:32:00</td>
      <td>05:32:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1-1-5300</td>
      <td>4</td>
      <td>4765</td>
      <td>05:33:00</td>
      <td>05:33:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1-1-5300</td>
      <td>5</td>
      <td>4773</td>
      <td>05:33:00</td>
      <td>05:33:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save the data frame
# Change the path to your output desired address

stop_times.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/stop_times.txt", index = False)
```

# Shapes.txt


```python
# Create a vector with length = number of shapes to iterate later

nshapes = list(range(0,len(rtrans.GID.unique())))
len(nshapes)
```




    1220




```python
# Create an empty DataFrame

shapes = pd.DataFrame(columns = ['shape_pt_lon', 'shape_pt_lat', 'shape_pt_sequence', 'shape_id'] )
```


```python
# Fill the DataFrame with the points for each of the shapes

for n in nshapes:
    df = pd.DataFrame(data = list(rtrans.geometry.iloc[n].coords), columns = ('shape_pt_lon', 'shape_pt_lat')) 
    df['shape_pt_sequence'] = pd.DataFrame(data = list(range(1,len(df)+1)))
    df['shape_id'] = rtrans.GID[n]
    shapes = shapes.append(df)   
```


```python
# Save the data frame
# Change the path to your output desired address

shapes.to_csv("/Users/santiagotoso/GoogleDrive/Master/Python/Montevideo/output/shapes.txt", index = False)
```
