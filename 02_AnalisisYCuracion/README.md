# Análisis y Curación de Datos

En este readme describimos el proceso de limpienza que realizamos sobre el dataset con los precios de ventas de propiedades en Melbourne, Australia. En particular vamos a utilizar este [dataset](https://cs.famaf.unc.edu.ar/~mteruel/datasets/diplodatos/melb_data.csv) producido por [DanB](https://www.kaggle.com/dansbecker)
Los pasos están en el mismo orden en el que los hicimos nosotros.

## Criterios de exclusión de columnas

1. `date`: esos años tuvieron solo 2% de inflación por lo que no cambia el precio
2. `seller`: Hay 268 vendedores/inmobiliarias en todo el DataSet. De estos 268, el 66% tiene muy pocas ventas, y el 30% sólamente 1 venta.
3. `method`: Creemos que no existe una relación entre el metodo de venta y el precio de la casa
4. Consideramos que el precio de un inmueble depende mucho de su ubicación (existe una frase americana que indica, los 3 factores que más influencian son: Location, Location, Location. Debido a esto, utilizaremos el indicador más preciso posible respecto a la zona de ubicación del inmueble. En este dataset, el agrupador por zona más preciso es `suburb`. Por eso, dropearemos las demás columnas que dan info geográfica:
    * `region_name`
    * `region_name_cat`
    * `council_area`
    * `council_area_cat`
    * `postcode`
    * `address`
5. `property_count`: Esto depende del suburb y la info queda repetida
6. `bedroom_2`: Coincide en más de un 90% con la col `rooms` y como viene de un scraper no es tan confiable

## Criterios de exclusión de filas

1. Casas con un precio mayor a 4e6 (55 registros)
2. Casas con un precio menor a 3900000 (124 registros)
3. Casas con un año de construción mayor a 1850 (2 registros)
4. Casas con un tamaño de terreno mayor a 882 pero incluyendo los nan (882 registros)
5. Cases con un tamaño de terreno menor a 400 pero incluyendo los nan (90 resgistros)

## Agrupaciones

La columna `suburb` nos parece relevante de analizar, pero para algunos valores tenemos muy pocos datos. Ya que tenemos las coordenadas geográficas, los vamos a agrupar por cercanía utilizando la [distancía Haversine](https://en.wikipedia.org/wiki/Haversine_formula) hasta que tengamos un mínimo de 5 registros por subirbio. La función de la distancia la implementamos nosotros, pero pudimos haber utilizado la de [scikit](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise.haversine_distances.html#:~:text=The%20Haversine%20(or%20great%20circle,the%20longitude%2C%20given%20in%20radians.)

## Datos agregados

Vamos a utilizar el dataset de airbnb para agregar datos que nos ayuden a entender el valor promedio en las cercanias a la casa en venta. Si bien son datos distintos (precio de compra de una casa a precio de alquiler por día) la hipotesis es que si el promedio de alquiler díaria es mas elevado el precio de vente de la casa va a ser más elevado.
Para cada casa en venta agregamos el promedio del precio y de review para las casas de airbnb que estan a una distancia de 1, 2.5 y 5km

## Transformaciones e Imputaciones

Realizamos dos imputaciones que parten del mismo df, pero selecionando un subconjunto de columnas

### Sin columnas categóricas

1. Descartamos las columnas `type`, `suburb_grouped`, `council_area`
2. Escalamos las columnas numericas usando `MinMaxScaler`
3. Imputamos los valores faltantes de `year_built` y `building_area` usando `KNeighborsRegressor`
4. Aplicamos la inversa del scaler para obtener nuevamente el df

### Con todas las columnas

1. Quisimos realizar una imputaciones con todas las columnas, pero como `suburb_grouped` tiene muchos valores únicos (unos 300 aprox) nos quedabamos sin RAM en colab por lo que la tuvimos que descartar
2. En las otras columnas categóricas (`type` y `council_area`) realizamos un `OneHotEncoding`
3. Escalamos las columnas numericas usando `MinMaxScaler`
4. Imputamos los valores faltantes de `year_built` y `building_area` usando `KNeighborsRegressor`
5. Computamos el PCA y sumamos a nuestro df las dos primeras dos componentes principales
6. Aplicamos la inversa del scaler para obtener nuevamente el df
