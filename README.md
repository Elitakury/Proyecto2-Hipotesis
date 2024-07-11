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
* Lenguaje SQL en BigQuery.
* Lenguaje Python en PowerBi.
 
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
# Procesamiento y preparación de datos:

Previo al análisis, importamos el dataset a un nuevo proyecto en BigQuery para proceder con la depuración de los datos. 

* Se identificó valores nulos y valores duplicados, además de carácteres raros.

* No se tomaron en cuenta los siguientes id: '5080031', '3814670', '8173823', '1119309', '0:00'.

``` sql
WITH solo_artist_songs AS (
 SELECT
   artist_s__name AS artist_name,
   COUNT(track_id) AS total_songs
 FROM `proyecto-2-hipotesis-426821.dataset_hipotesis.track_in_spotify`
 WHERE
   artist_count = 1
 GROUP BY
   artist_s__name
)
SELECT
 T1.track_id,
 REGEXP_REPLACE(T1.track_name, r'[^a-zA-Z0-9\s]', ' ') AS track_name,
 REGEXP_REPLACE(T1.artist_s__name, r'[^a-zA-Z0-9\s]', ' ') AS artist_s__name,
 T1.artist_count,
 T1.released_year,
 T1.released_month,
 T1.released_day,
 CAST(CONCAT(CAST(T1.released_year AS STRING), '-',
 LPAD(CAST(T1.released_month AS STRING), 2, '0'), '-',
 LPAD(CAST(T1.released_day AS STRING), 2, '0')) AS DATE) AS fecha_released,
 T1.in_spotify_playlists,
 T1.in_spotify_charts,
 SAFE_CAST(IF(REGEXP_CONTAINS(T1.streams, r'^[0-9]+$'), T1.streams, NULL) AS INT64) AS streams_int64,
 T1.in_spotify_playlists + T2.in_apple_playlists + T2.in_deezer_playlists AS total_playlists,
 SAS.total_songs AS solo_artist_total_songs
FROM
 `proyecto-2-hipotesis-426821.dataset_hipotesis.track_in_spotify` AS T1
JOIN
 `proyecto-2-hipotesis-426821.dataset_hipotesis.track_in_competition` AS T2
 ON T1.track_id = T2.track_id
LEFT JOIN
 solo_artist_songs AS SAS
 ON T1.artist_s__name = SAS.artist_name
WHERE
 T1.track_id NOT IN ('5080031', '3814670', '8173823', '1119309', '0:00', '4061483', '5865058','6720570');
```

* Se identificaron que las variables key y mode dentro de la tabla de technical info no son útiles dentro del análisis. 

* Se identifivó y modificó la variable streams la cual estaba en formato texto a formato número. 

* Se realizó un consolidado junto con las nuevas variables usando LEFT JOIN y la vista (view) con los datos limpios de cada tabla.

``` sql
CREATE OR REPLACE TABLE `proyecto-2-hipotesis-426821.dataset_hipotesis.consolidado` AS
SELECT
S.track_id,
S.track_name,
S.artist_s__name,
S.artist_count,
S.released_day,
S.released_month,
S.released_year,
S.fecha_released,
S.in_spotify_charts,
S.in_spotify_playlists,
S.streams_int64,
S.total_playlists,
C.in_apple_charts,
C.in_apple_playlists,
C.in_deezer_charts,
C.in_deezer_playlists,
C.in_shazam_charts,
T.bpm,
T.`acousticness_%`,
T.`danceability_%`,
T.`energy_%`,
T.`instrumentalness_%`,
T.`liveness_%`,
T.`speechiness_%`,
T.`valence_%`,
Q.bpm_category,
Q.streams_category,
Q.acousticness_category,
Q.danceability_category,
Q.energy_category,
Q.instrumentalness_category,
Q.liveness_category,
Q.speechiness_category,
Q.valence_category
FROM `proyecto-2-hipotesis-426821.dataset_hipotesis.view_track_in_spotify_clean` AS S
LEFT JOIN
`proyecto-2-hipotesis-426821.dataset_hipotesis.view_in_competition_clean` AS C
ON
S.track_id = C.track_id
LEFT JOIN
 `proyecto-2-hipotesis-426821.dataset_hipotesis.view_track_technical_info_clean` AS T
ON
S.track_id = T.track_id
LEFT JOIN
 `proyecto-2-hipotesis-426821.dataset_hipotesis.categoria` AS Q
ON
 T.track_id = Q.track_id
ORDER BY
 S.artist_s__name,
 S.released_year,
 S.released_month,
 S.released_day;
```

* Se crearon nuevas variables:

1. fecha_released: concatenando día, mes y año. 

2. total_playlist: sumando in_spotify_playlists e in_spotify_charts.

3. season: estaciones del año.

4. Cuartiles y categoría: Se asignó según la segmentación de cuartiles la categoría "alto" y "bajo" a cada característica de canción de la tabla technical info de la siguiente forma:

``` sql
CREATE OR REPLACE VIEW `proyecto-2-hipotesis-426821.dataset_hipotesis.quartiles` AS
  SELECT
	track_id,
	streams_int64,
	bpm,
	`acousticness_%`,
	`danceability_%`,
	`energy_%`,
	`instrumentalness_%`,
	`liveness_%`,
	`speechiness_%`,
	`valence_%`,
	NTILE(4) OVER (ORDER BY bpm) AS q_bpm,
	NTILE(4) OVER (ORDER BY streams_int64) AS q_streams,
	NTILE(4) OVER (ORDER BY `danceability_%`) AS q_danceability,
	NTILE(4) OVER (ORDER BY `valence_%`) AS q_valence,
	NTILE(4) OVER (ORDER BY `energy_%`) AS q_energy,
	NTILE(4) OVER (ORDER BY `acousticness_%`) AS q_acousticness,
	NTILE(4) OVER (ORDER BY `instrumentalness_%`) AS q_instrumentalness,
	NTILE(4) OVER (ORDER BY `liveness_%`) AS q_liveness,
	NTILE(4) OVER (ORDER BY `speechiness_%`) AS q_speechiness
  FROM
	`proyecto-2-hipotesis-426821.dataset_hipotesis.consolidado`
  WHERE streams_int64 IS NOT NULL
```
```sql
CREATE OR REPLACE VIEW `proyecto-2-hipotesis-426821.dataset_hipotesis.categoria` AS
SELECT
  a.track_id,
  a.streams_int64,
  a.bpm,
  a.`acousticness_%`,
  a.`danceability_%`,
  a.`energy_%`,
  a.`instrumentalness_%`,
  a.`liveness_%`,
  a.`speechiness_%`,
  a.`valence_%`,
  q.q_bpm,
  q.q_streams,
  q.q_danceability,
  q.q_valence,
  q.q_energy,
  q.q_acousticness,
  q.q_instrumentalness,
  q.q_liveness,
  q.q_speechiness,
  CASE
	WHEN q.q_bpm IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_bpm = 4 THEN "Alto"
  END AS bpm_category,
  CASE
	WHEN q.q_streams IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_streams = 4 THEN "Alto"
  END AS streams_category,
  CASE
	WHEN q.q_danceability IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_danceability = 4 THEN "Alto"
  END AS danceability_category,
  CASE
	WHEN q.q_valence IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_valence = 4 THEN "Alto"
  END AS valence_category,
  CASE
	WHEN q.q_energy IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_energy = 4 THEN "Alto"
  END AS energy_category,
  CASE
	WHEN q.q_acousticness IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_acousticness = 4 THEN "Alto"
  END AS acousticness_category,
  CASE
	WHEN q.q_instrumentalness IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_instrumentalness = 4 THEN "Alto"
  END AS instrumentalness_category,
  CASE
	WHEN q.q_liveness IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_liveness = 4 THEN "Alto"
  END AS liveness_category,
  CASE
	WHEN q.q_speechiness IN (1, 2, 3) THEN "Bajo"
	WHEN q.q_speechiness = 4 THEN "Alto"
  END AS speechiness_category
FROM
  `proyecto-2-hipotesis-426821.dataset_hipotesis.consolidado` a
LEFT JOIN `proyecto-2-hipotesis-426821.dataset_hipotesis.quartiles` q
ON a.track_id = q.track_id
WHERE a.streams_int64 IS NOT NULL;
```

# Visualización y análisis de datos
* Importamos datos desde BigQuery hacia PowerBi.

* Se realizó ténicas de agrupación según variables categóricas, obteniendo la cantidad de canciones por artista y por año de lanzamiento. Obtenemos la siguiente tabla:

![alt text](imagen1.png)

* Se aplicó medidas de tendencia central con la ayuda del lenguaje de Python para visualizar distribución a través de un histograma:

  * Histograma BPM:
  Muestra la distribución de BPM en el conjunto de datos. Los BPM varían entre 75 y 200, con una mayor concentración en los rangos más bajos. Esto sugiere que las canciones con BPM más lentos son más comunes en el conjunto de datos.

![alt text](imagen2.png)

 * Histograma Streams:
 Muestra la frecuencia en la cantidad de veces que una canción fue escuchada. En su mayoría están concentradas en el primer intervalo (lado izquierdo) esto sugiere que 

![alt text](imagen3.png)

# -Presentación de resultados.

Validación de hipótesis 
# * Hipótesis 1: 
Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify 

![alt text](imagen4.png)

Respuesta: Se rechaza la hipótesis de que las canciones con un mayor BPM tienen más éxito en términos de cantidad de streams en Spotify, ya que el valor de la correlación obtenido (-0.002336203586) indica que no existe una relación significativa entre estas variables.

# Hipótesis 2: 
Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.

![alt text](imagen5.png)
![alt text](imagen6.png)
![alt text](imagen7.png)

Respuesta:  La hipótesis es aceptada, porque los valores de las correlaciones, que son 0.5999860553480 para Deezer, 0.5515564831921 para Apple Music, y 0.605488541573 para Shazam, indican una relación moderada a fuerte, lo que sugiere que las canciones populares en Spotify tienden a ser populares también en estas otras plataformas.

# Hipótesis 3: 
La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.

![alt text](imagen8.png)

Respuesta: El valor de la correlación obtenido (0.783568569951) indica una relación fuerte y positiva, sugiriendo que las canciones que aparecen en más playlists tienden a tener más streams.

# Hipótesis 4: 
Los artistas con un mayor número de canciones en Spotify tienen más streams.

![alt text](imagen9.png)

Respuesta: Dado que el valor de la correlación es alto (0.77892616317456), podemos concluir que existe una relación fuerte y positiva entre el número de canciones que un artista tiene en Spotify y la cantidad total de streams que recibe. Esto sugiere que los artistas con más canciones tienden a tener más streams.

# Hipótesis 5: 
Las características de la canción influyen en el éxito en términos de cantidad de streams en Spotify.

 * Correlación streams vs danceability: -0.105501164099

![alt text](imagen10.png)

 * Correlación streams con valency: -0.041370563742

 ![alt text](imagen11.png)

 * Correlación streams con acousticness: -0.0052737047472

 ![alt text](imagen12.png)

 * Correlación streams con energy: -0.025626822823

 ![alt text](imagen13.png)

 * Correlación streams con instrumentalnes: -0.04423444890059

 ![alt text](imagen14.png)

 * Correlación streams con liveness: -0.049480680490

 ![alt text](imagen15.png)

 * Correlacióntreams con speechiness: -0.112763169712

![alt text](imagen16.png)

Respuesta: Los valores de las correlaciones obtenidos son todos negativos y muy bajos, lo cual indica que no existe una relación significativa entre estas características de las canciones y su éxito en términos de cantidad de streams en Spotify. Ninguna de las características evaluadas parece tener un impacto considerable en el número de streams.

# Conclusiones:

* Popularidad en Múltiples Plataformas: Las canciones que son populares en Spotify tienden a ser populares en otras plataformas como Deezer, Apple Music y Shazam. Esto sugiere que una estrategia de promoción que abarque múltiples plataformas puede ser efectiva para aumentar el éxito de una canción.

* La presencia de una canción en un mayor número de playlists tiene una relación fuerte y positiva con la cantidad de streams. Esto indica que una estrategia de inclusión en playlists, tanto oficiales como de usuarios, es crucial para aumentar los streams.

* Los artistas con un mayor número de canciones en Spotify tienden a tener más streams en total. Esto sugiere que una mayor producción y publicación de contenido puede contribuir al éxito en términos de streams.

# Recomendaciones:


# Enlaces de interés:

