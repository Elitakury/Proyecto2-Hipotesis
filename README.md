# Proyecto2-Hipotesis
# Contexto: 
Una discográfica se enfrenta al emocionante desafío de lanzar un nuevo artista en el escenario musical global.
# Objetivo:
Explorar un conjunto de datos con el fin de identificar patrones o características que puedan determinar los factores que contribuyen al éxito de una canción teniendo como herramienta 
un extenso dataset de Spotify con información sobre las canciones más escuchadas en 2023.
Las respuestas obtenidas nos ayudarán a realizar la validación de las siguientes hipótesis:
1.Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.
2.Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.
3.La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.
4.Los artistas con un mayor número de canciones en Spotify tienen más streams.
5.Las características de la canción influyen en el éxito en términos de cantidad de streams en Spotify.

# Equipo:
Trabajo realizado de forma grupal.
Elizabeth Takury y Natalia Alejandro. 

# Herramientas y Tecnologías:
# 1.Herramientas:
* Google Sheets
* BigQuery
* Power BI
* ChatGPT 
* Google Slides 
* Loom
# 2.Lenguajes:
*Lenguaje SQL en BigQuery.
 
# Descripción de las variables del dataset
# Trackinspotify
- track_id: Identificador único de la canción. Es un número entero de 7 dígitos que no se repite

- track_name: Nombre de la canción

- artist(s)_name: Nombre del artista(s) de la canción

- artist_count: Número de artistas que contribuyen a la canción.

- released_year: Año en que se lanzó la canción.

- released_month: Mes en el que se lanzó la canción.

- released_day: Día del mes en que se lanzó la canción.

- inspotifyplaylists: Número de listas de reproducción de Spotify en las que está incluida la canción

- inspotifycharts: Presencia y ranking de la canción en las listas de Spotify

- streams: Número total de transmisiones en Spotify. Representa la cantidad de veces que la canción fue escuchada.
# Trackincompetition

- track_id: Identificador único de la canción. Es un número entero de 7 dígitos que no se repite

- inappleplaylists: número de listas de reproducción de Apple Music en las que está incluida la canción

- inapplecharts: Presencia y rango de la canción en las listas de Apple Music

- indeezerplaylists: Número de listas de reproducción de Deezer en las que está incluida la canción

- indeezercharts: Presencia y rango de la canción en las listas de Deezer

- inshazamcharts: Presencia y rango de la canción en las listas de Shazam
# Tracktechnicalinfo
- track_id: Identificador único de la canción. Es un número entero de 7 dígitos que no se repite

- bpm: Pulsaciones por minuto, una medida del tiempo de la canción.

- key: Clave musical de la canción

- mode: Modo de la canción (mayor o menor)

- danceability_%: Porcentaje que indica qué tan adecuada es la canción para bailar

- valence_: Positividad del contenido musical de la canción.

- energy_: Nivel de energía percibido de la canción.

- acusticness_: Cantidad de sonido acústico en la canción.

- instrumentality_: Cantidad de contenido instrumental en la canción.

- liveness_: Presencia de elementos de actuación en vivo.

- speechiness_: Cantidad de palabras habladas en la canción.
# Procesamiento y análisis:
# -Procesar y preparar base de datos.
Previo al análisis, importamos el dataset a un nuevo proyecto en BigQuery para proceder con la depuración de los datos. 

* Se identificó valores nulos y valores duplicados, además de carácteres raros. 

* Se identificaron las variables key y mode dentro de la tabla de technical info que no son útiles dentro de mi análisis. 

* Se identifivó y modificó la variable streams la cual estaba en formato texto pero debería estar en formato número. 

* Se crearon nuevas variables:
1. fecha_released: concatenando día, mes y año. 
2. totak_playlist: sumando in_spotify_playlists e in_spotify_charts.

* Se realizó un consolidado junto con las nuevas variables usando LEFT JOIN y la vista (view) con los datos limpios de cada tabla.

# -Hacer un análisis exploratorio (AED).
1.  Agrupar datos según variables categóricas.
2.  Agrupar variables por categorías.
3.  Aplicar medidas de tendencia central.
4.  Visualizar distribución.
5.  Aplicar medidas de dispersión.
6.  Visualizar el comportamiento de los datos a lo largo del tiempo.
7.  Calcular cuartiles, deciles o percentiles.
8.  Calcular correlación entre variables.
# -Aplicar técnicas de análisis
1. 
# -Construcción de dashboard.

# -Presentación de resultados.

# Resultados: 
# Conclusiones:

![alt text](Imagen1.png)

# Enlaces de interés:

