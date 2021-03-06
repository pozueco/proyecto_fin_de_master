Creación del recomendador de películas
======================================

Leemos los datos desde el fichero de películas original:

``` r
fileMovies <- file("./movie_metadata_original.csv","r") 
moviesDataset <- read.csv(fileMovies) 
close(fileMovies) 
head(moviesDataset[2827,1:20])
```

Limpiamos los datos y nos quedamos sólo con aquellas columnas utilizadas por el recomendador:

``` r
# new data frame only with selected columns
# filter columns from South Korea and Japan
moviesRecommender <- moviesDataset[moviesDataset[,21] == "USA",]

moviesRecommender <- subset(moviesDataset, select=c("imdb_score",
                                                    "genres",
                                                    "movie_title",
                                                    "title_year",
                                                    "actor_1_name",
                                                    "actor_2_name",
                                                    "actor_3_name",
                                                    "director_name"))

# filter columns with NA values
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,1]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,2]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,3]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,4]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,5]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,6]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,7]),]
moviesRecommender <- moviesRecommender[!is.na(moviesRecommender[,8]),]

# create new columns for each genre
moviesRecommender$drama_genre = ifelse(grepl("Drama",moviesRecommender$genres), 1, 0)
moviesRecommender$comedy_genre = ifelse(grepl("Comedy",moviesRecommender$genres), 1, 0)
moviesRecommender$thriller_genre = ifelse(grepl("Thriller",moviesRecommender$genres), 1, 0)
moviesRecommender$action_genre = ifelse(grepl("Action",moviesRecommender$genres), 1, 0)
moviesRecommender$romance_genre = ifelse(grepl("Romance",moviesRecommender$genres), 1, 0)
moviesRecommender$adventure_genre = ifelse(grepl("Adventure",moviesRecommender$genres), 1, 0)
moviesRecommender$crime_genre = ifelse(grepl("Crime",moviesRecommender$genres), 1, 0)
moviesRecommender$scifi_genre = ifelse(grepl("Sci-Fi",moviesRecommender$genres), 1, 0)
moviesRecommender$fantasy_genre = ifelse(grepl("Fantasy",moviesRecommender$genres), 1, 0)
moviesRecommender$horror_genre = ifelse(grepl("Horror",moviesRecommender$genres), 1, 0)
moviesRecommender$family_genre = ifelse(grepl("Family",moviesRecommender$genres), 1, 0)
moviesRecommender$mystery_genre = ifelse(grepl("Mystery",moviesRecommender$genres), 1, 0)
moviesRecommender$biography_genre = ifelse(grepl("Biography",moviesRecommender$genres), 1, 0)
moviesRecommender$animation_genre = ifelse(grepl("Animation",moviesRecommender$genres), 1, 0)
moviesRecommender$music_genre = ifelse(grepl("Music",moviesRecommender$genres), 1, 0)
moviesRecommender$war_genre = ifelse(grepl("War",moviesRecommender$genres), 1, 0)
moviesRecommender$history_genre = ifelse(grepl("History",moviesRecommender$genres), 1, 0)
moviesRecommender$sport_genre = ifelse(grepl("Sport",moviesRecommender$genres), 1, 0)
moviesRecommender$musical_genre = ifelse(grepl("Musical",moviesRecommender$genres), 1, 0)
moviesRecommender$documentary_genre = ifelse(grepl("Documentary",moviesRecommender$genres), 1, 0)
moviesRecommender$western_genre = ifelse(grepl("Western",moviesRecommender$genres), 1, 0)
moviesRecommender$filmnoir_genre = ifelse(grepl("Film-Noir",moviesRecommender$genres), 1, 0)
moviesRecommender$short_genre = ifelse(grepl("Short",moviesRecommender$genres), 1, 0)
moviesRecommender$news_genre = ifelse(grepl("News",moviesRecommender$genres), 1, 0)
```

En nuestro recomendador, en base al título de la película buscaremos las pel?culas que tengan en común algún actor o el director, y calcularemos la similitud con nuestra película en base a la puntuación en IMDb y el género de las películas:

``` r
# title of the film to recommend
movie = "The Matrix"
vec1 = subset(moviesRecommender, moviesRecommender$movie_title==movie)[1,]

# get films with the same genres
moviesRecommenderFiltered <- c()
for(p in c(toString(vec1$actor_1_name), 
           toString(vec1$actor_2_name), 
           toString(vec1$actor_3_name), 
           toString(vec1$director_name))){
  print(p)
  
  moviesRecommenderFiltered <- rbind(moviesRecommenderFiltered,
                                     subset(moviesRecommender, grepl(p, moviesRecommender$actor_1_name)))
  moviesRecommenderFiltered <- rbind(moviesRecommenderFiltered,
                                     subset(moviesRecommender, grepl(p, moviesRecommender$actor_2_name)))
  moviesRecommenderFiltered <- rbind(moviesRecommenderFiltered,
                                     subset(moviesRecommender, grepl(p, moviesRecommender$actor_3_name)))
  moviesRecommenderFiltered <- rbind(moviesRecommenderFiltered,
                                     subset(moviesRecommender, grepl(p, moviesRecommender$director_name)))
}
```

    ## [1] "Keanu Reeves"
    ## [1] "Marcus Chong"
    ## [1] "Gloria Foster"
    ## [1] "Lana Wachowski"

``` r
# prepare films to recommend
col = colnames(moviesRecommenderFiltered)[9:length(colnames(moviesRecommenderFiltered))]
col = c("imdb_score", col)
features = moviesRecommenderFiltered[,col]

# search similar films
sim <- apply(features, 1, function(x) sum(vec1[,col] * x,na.rm=T)/(sqrt(sum(vec1[,col]^2,na.rm=T))*sqrt(sum(x^2,na.rm=T))))

# show recommendations
data <- cbind(moviesRecommenderFiltered, sim)
data=data[order(data$sim, data$movie_title, decreasing = T),]
data=subset(data,movie_title!=movie)

subset(data, select=c("imdb_score", "movie_title", "title_year", "sim"))
```

    ##      imdb_score                       movie_title title_year       sim
    ## 126         7.2               The Matrix Reloaded       2003 0.9994619
    ## 124         6.7            The Matrix Revolutions       2003 0.9989014
    ## 62          5.4                 Jupiter Ascending       2015 0.9798920
    ## 4177        7.1              My Own Private Idaho       1991 0.9773975
    ## 4822        8.1                 Nothing But a Man       1964 0.9723357
    ## 2012        7.7                Dangerous Liaisons       1988 0.9708063
    ## 2771        7.7                Dangerous Liaisons       1988 0.9708063
    ## 4330        7.1                      River's Edge       1986 0.9680281
    ## 876         5.6                    Chain Reaction       1996 0.9676996
    ## 3452        7.0                    The Neon Demon       2016 0.9674970
    ## 232         6.1                       Speed Racer       2008 0.9674015
    ## 1587        7.2                             Speed       1994 0.9662175
    ## 1277        6.7                    Sweet November       2001 0.9657648
    ## 975         6.5                  The Replacements       2000 0.9644804
    ## 3100        6.9  Bill & Ted's Excellent Adventure       1989 0.9638154
    ## 2132        6.3                          Hardball       2001 0.9630776
    ## 2221        6.8                      Street Kings       2008 0.9629427
    ## 825         7.5              The Devil's Advocate       1997 0.9617314
    ## 1201        7.5             Bram Stoker's Dracula       1992 0.9617314
    ## 466         5.5     The Day the Earth Stood Still       2008 0.9611390
    ## 4474        5.5     The Day the Earth Stood Still       2008 0.9611390
    ## 3367        7.4            Much Ado About Nothing       1993 0.9610696
    ## 3942        5.8 The Last Time I Committed Suicide       1997 0.9589497
    ## 85          6.3                          47 Ronin       2013 0.9579403
    ## 2307        7.1                  A Scanner Darkly       2006 0.9566994
    ## 1239        6.8                    The Lake House       2006 0.9565035
    ## 450         6.7            Something's Gotta Give       2003 0.9556285
    ## 3838        7.4                             Bound       1996 0.9528565
    ## 2194        6.2        Bill & Ted's Bogus Journey       1991 0.9457170
    ## 1529        5.3                       The Watcher       2000 0.9234804
