Limpieza y preparación de datos
-------------------------------

En primer lugar se cargan los datos se desde el directorio actual, almacenados en un fichero CSV:

``` r
fileMovies <- file("./movie_metadata_original.csv","r") 
moviesDataset <- read.csv(fileMovies) 
close(fileMovies) 
head(moviesDataset[,1:5])
```

Seleccioneamos únicamente las columnas que serán tenidas en cuenta en el modelo no supervisado:

``` r
# new data frame only with selected columns
moviesDatasetClustering <- subset(moviesDataset, select=c("actor_1_facebook_likes", 
                                                          "actor_2_facebook_likes", 
                                                          "actor_3_facebook_likes",
                                                          "director_facebook_likes"))

# filter columns with NA values
moviesDatasetClustering <- moviesDatasetClustering[!is.na(moviesDatasetClustering[,1]),]
moviesDatasetClustering <- moviesDatasetClustering[!is.na(moviesDatasetClustering[,2]),]
moviesDatasetClustering <- moviesDatasetClustering[!is.na(moviesDatasetClustering[,3]),]
moviesDatasetClustering <- moviesDatasetClustering[!is.na(moviesDatasetClustering[,4]),]
```

Guardamos los datos procesados en un nuevo fichero para utilizarlos en el modelo no supervisado y se eliminan las filas con datos no definidos:

``` r
write.csv(moviesDatasetClustering, "./movie_metadata_clean_no_supervised.csv") 
```
