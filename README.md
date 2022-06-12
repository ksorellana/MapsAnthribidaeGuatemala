# Anthribidae of Guatemala distribution map in R
# Mapa de distribución de Anthribidae de Guatemala en R
K. Samanta Orellana, 02 jan 2022


**Please cite**: Orellana, K.S. (2022). MapsAnthribidaeGuatemala. GitHub Repository. [https://doi.org/10.5281/zenodo.6636379](https://zenodo.org/badge/latestdoi/470294287)

[![DOI](https://zenodo.org/badge/470294287.svg)](https://zenodo.org/badge/latestdoi/470294287)

Distribution map of occurrence records of Anthribidae in Guatemala up to January 2002.
Mapa de distribución de registros de ocurrencia de Anthribidae hasta enero de 2022. 

### Construcción de mapas con ggplot2, sf y tidyverse
References/Referencias: 
- https://github.com/wtesto/SpeciesOccurrenceMapping
- https://r-spatial.org/r/2018/10/25/ggplot2-sf.html
- https://slcladal.github.io/maps.html

### Capas adicionales descargadas de
- DivaGis: https://www.diva-gis.org/gdata (divisiones administrativas: GTM_adm0.shp (país), GTM_adm1.shp (departamentos), GTM_adm2.shp (municipios))

### Paquetes necesarios
Instalar paquetes

```
install.packages("tidyverse")
install.packages("sf")
install.packages("ggplot2)
install.packages("raster")
install.packages("rnaturalearth")
install.packages("sf")
install.packages("elevatr")
install.packages("dplyr")
install.packages("magrittr")
install.packages("ggspatial")
install.packages("ggplot2")
install.packages("ggpubr")
```

Instalar "rnaturalearthhires" desde github para que no de problemas

```
install.packages("githubinstall")
library(githubinstall)
githubinstall("rnaturalearthhires")
```

Cargar paquetes

```
library(tidyverse)
library(sf)
library(ggplot2)
library(raster) #for processing some spatial data
library(rnaturalearth) #for downloading shapefiles
library(sf) #for processing shapefiles
library(elevatr) #for downloading elevation data
library(dplyr) #to keep things tidy
library(magrittr) #to keep things very tidy
library(ggspatial) #for scale bars and arrows
library(ggplot2) #for tidy plotting
library(ggpubr) #for easy selection of symbols
library(rnaturalearthhires)
```

Cargar capas que van a construir el mapa de fondo con ggplot

```
world <- ne_countries(scale = "medium", returnclass = "sf") #Mundo, para construir los mapas
```

Para definir el área del mapa

```
library(rnaturalearthhires)
library(raster) #for processing some spatial data
map <- ne_countries(scale = 10, returnclass = "sf")
```

Cargar las capas del mapa que quieran ser utilizadas

```
map <- ne_countries(scale = 10, returnclass = "sf")
states <- ne_states(returnclass = "sf")
ocean <- ne_download(scale = 10, type = 'ocean', 
                   category = 'physical', returnclass = 'sf')
rivers <- ne_download(scale = 10, type = 'rivers_lake_centerlines', 
                    category = 'physical', returnclass = 'sf')
```

Para definir el área del mapa, en este caso, Guatemala

```
focalArea <- map %>% filter(admin == "Guatemala")
limit <- st_buffer(focalArea, dist = 1) %>% st_bbox()
clipLimit <- st_buffer(focalArea, dist = 2) %>% st_bbox()
limitExtent <- as(extent(clipLimit), 'SpatialPolygons')
crs(limitExtent) <- "+proj=longlat +datum=WGS84 +no_defs"
```

### Capas de Guatemala

-Descargar capa de departamentos en DivaGis (https://www.diva-gis.org/gdata) si no quiere usarse la capa de "states" que incluye todas las divisiones de los otros países.
-DivaGis: https://www.diva-gis.org/gdata
-Guardar todos los archivos en el directorio que estemos usando

Abrir directorio donde guardamos y extrajimos las capas

```
setwd("C://Coding")
```

Cargar capas de Guatemala

```
gua_pais <- st_read("GTM_adm0.shp")
gua_dep <- st_read("GTM_adm1.shp")
gua_mun <- st_read("GTM_adm2.shp")
gua_rios <- st_read("GTM_water_lines_dcw.shp")
gua_lagos <- st_read("GTM_water_areas_dcw.shp")
```

### Puntos de ocurrencia
Cargar el (los) archivo(s) con los puntos (especie -u otra variable-, lat, long)

Abrir directorio

```
setwd("C://Coding")
```

Añadir puntos con archivos .csv, con tres columnas: especie (o alguna otra variable), latitud y longitud

```
anthrigt <- read.csv("anthrgtreg.csv")
```

Revisar los nombres de las columnas

```
names(anthrigt)
head(anthrigt)
```
## Mapa de Guatemala con departamentos y coloreado por número de especies

Cargar la capa que se va a usar para colorear, en este caso departamentos

```
setwd("C://Coding")
gua_dep <- st_read("GTM_adm1.shp")
```

Cargar el archivo .csv con las columnas de departamentos (NAME_1) y número de especies
Que la columna de depatamentos se llame igual y que todos los nombres coincidan

```
especies <- read_csv("especies.csv")
especies
names(especies)
```

Unir esta tabla al archivo del mapa (asegurarse que el nombre de las columnas a unir sean iguales (NAME_1 = NAME_1)

```
gua_dep_ant <- gua_dep %>%
	left_join(especies)
```	
  
	###Si las columnas tienen nombres distintos, se puede hacer esto, por ejemplo:
```
  left_join(especies,
            by = c("NAME_1" = "departamento"))
```

Para revisar la tabla obtenida

```
gua_dep_ant
names(gua_dep_ant)

ggplot(data = world) +
  geom_sf(fill="white")+
	geom_sf(data = ocean, color = "blue", size = 0.05, fill = "#add8e6") +
	geom_sf(data = focalArea, color = "black", size = 0.15,
          linetype = "solid", fill = "white", alpha = 0.5) +
	geom_sf(data=gua_dep, fill="white") +
	geom_sf(data=gua_dep_ant, aes(fill=especies)) +
	scale_fill_gradient ("Total de especies", high = "#A4C61A", low = "white") +
	 	labs( x = "Longitud", y = "Latitud") +
  coord_sf(xlim = c(-92.5, -88.1), ylim = c(13.8, 18.1), expand = T) +
  annotation_scale(location = "bl", width_hint = 0.3) +
  annotation_north_arrow(location = "bl", which_north = "true", 
                         pad_x = unit(0.75, "in"), pad_y = unit(0.3, "in"),
                         style = north_arrow_fancy_orienteering) +
  theme_bw()
```

Para guardar el mapa

```
ggsave("AnthribidaeGuatemalaDepartamentoEspecies.jpg")
```

## Mapa de Guatemala con registros de ocurrencia

```
ggplot(data = world) +
  geom_sf(fill="white")+
	geom_sf(data = ocean, color = "blue", size = 0.05, fill = "aliceblue") +
	geom_sf(data = focalArea, color = "black", size = 0.15,
          linetype = "solid", fill = "white", alpha = 0.5) +
	geom_sf(data=gua_dep, fill="white") +
	geom_sf(data=gua_dep_ant, aes(fill=especies), linetype="solid", size=0.3) +
	scale_fill_gradient ("Species total", high = "#f5e267", low = "white") +
geom_point(data = anthrigt, aes(x=decimalLongitude, y = decimalLatitude, color=Record, pch=Record), cex = 2) + #esta es la línea donde van los puntos, nombrar adecuadamente las columnas de lon y lat
	 scale_color_manual(values=c("purple", "red", "black"))+
	 	labs( x = "Longitude", y = "Latitude") +
  coord_sf(xlim = c(-92.5, -88.1), ylim = c(13.8, 18.1), expand = T) +
  annotation_scale(location = "bl", width_hint = 0.3) +
  annotation_north_arrow(location = "bl", which_north = "true", 
                         pad_x = unit(0.75, "in"), pad_y = unit(0.3, "in"),
                         style = north_arrow_fancy_orienteering) +
  theme_bw()
```

Para guardar el mapa

```
ggsave("AnthribidaeGuatemalaPointsDeptoGreen.jpg")

```

Colores: amarillo #f5e267 verde #9cf536

| <img src="https://github.com/ksorellana/MapsAnthribidaeGuatemala/blob/main/files/AnthribidaeGuatemalaPorEspecieDepartamento.png?raw=true" alt="Mapa Departamentos" width="470" height="410"> <img src="https://github.com/ksorellana/MapsAnthribidaeGuatemala/blob/main/files/MapaFinalArt%C3%ADculo.jpg?raw=true" alt="Mapa Dep Puntos" width="490" height="400"> | 
|:--:| 
|Mapa de Guatemala departamentos coloreados y registros de ocurrencia de Anthribidae.|

