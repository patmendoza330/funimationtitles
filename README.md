2/21/2022

# Collecting Anime listings from various platforms - Funimation

I’ve created a [tableau
dashboard](https://public.tableau.com/app/profile/patrick.mendoza5877/viz/WhatAnimetoWatchNextMyAnimeList/Dashboard)
which is an exploratory view of top ranked and popular anime titles from
data at <http://myanimelist.net>. The purpose of this dashboard is to
assist the end-user in locating their next anime to watch. However, what
is lacking from this tool is the streaming platform that currently
houses the titles.

To add that information into the dashboard, I need to locate or scrape
the anime titles that are housed on various platforms.

The most popular platforms for viewing anime titles are:

-   [Crunchyroll](https://github.com/patmendoza330/crunchyrolltitles)
-   Funimation
-   Netflix
-   Hulu
-   Amazon Prime

This list is not all-inclusive and in no particular order.

**This code** will encompass downloading Funimation titles that have
been scraped from the MyAnimeList company page for Funimation at
<https://myanimelist.net/anime/producer/102/Funimation>

## First things first - setting some variables

Set your working directory to wherever you’d like in the
`WORKINDIRECTORY` section.

``` r
wd1 = WORKINGDIRECTORY
setwd(wd1)
```

## Install and load any libraries

The `tidyverse` contains the `rvest` library which will be used to
scrape the data, while `dplyr` and `tidyr` will be used to
manipulate/transform data.

``` r
install.packages("tidyverse")
```

Next, we want to load two libraries:

``` r
library(tidyverse)
library(rvest)
```

Although `rvest` is a component of the tidyverse, it doesn’t
automatically load with the library call `tidyverse`, as a result,
you’ll need to load it separately.

## Funimation

Funimation does have a listing of all of their titles on a webpage at:
<https://www.funimation.com/shows/>. But it does not allow scraping that
I am aware of.

The MyAnimeList website does have a page dedicated to Funimation which
carries other attributes relating to ranking and number of users that we
will be scraping at
<https://myanimelist.net/anime/producer/102/Funimation>.

### Scraping the Funimation Title List

``` r
funimation <- read_html('https://myanimelist.net/anime/producer/102/Funimation')

right_side_bucket <- funimation %>%
    html_element(xpath = '/html/body/div[1]/div[2]/div[3]/div[2]/div[2]')
```

I wasn’t able to identify a single container for the table that is used
for all of the titles, so I’ve selected the right side of the screen as
my starting point.

Grabbing all link information and any associated text strings between
those links will be my starting point.

``` r
# grab links
link <- right_side_bucket %>%
  html_elements('a') %>%
  html_attr('href')

# grab text between link
link_text <- right_side_bucket %>%
  html_elements('a') %>%
  html_text(trim = TRUE)

head(link)
```

    ## [1] "https://myanimelist.net/anime/producer/102/Funimation"     
    ## [2] "https://myanimelist.net/anime/producer/102/Funimation/news"
    ## [3] "https://myanimelist.net/"                                  
    ## [4] "https://myanimelist.net/company"                           
    ## [5] "https://myanimelist.net/anime/producer/102/Funimation"     
    ## [6] "https://myanimelist.net/anime/producer/102/Funimation/news"

``` r
head(link_text)
```

    ## [1] "Details"    "News"       "Top"        "Companies"  "Funimation"
    ## [6] "More News"

The structure of the website also stores the following within `span`
elements:

-   members - the number of members who have watched or are watching
    this title
-   score - the average score (from 1 to 10) from all users
-   start\_date - the start date of the anime
-   title - the title of the anime

I’ll grab those components by selecting the specific `classes` within
the `span` elements:

``` r
# grab members
members <- right_side_bucket %>%
  html_elements('span.js-members') %>%
  html_text(trim = TRUE)

# grab score
score <- right_side_bucket %>%
  html_elements('span.js-score') %>%
  html_text(trim = TRUE)

# grab Startdate
start_date <- right_side_bucket %>%
  html_elements('span.js-start_date') %>%
  html_text(trim = TRUE)

# grab title
anime_title <- right_side_bucket %>%
  html_elements('span.js-title') %>%
  html_text(trim = TRUE)

length(link)
```

    ## [1] 3614

``` r
length(link_text)
```

    ## [1] 3614

``` r
length(members)
```

    ## [1] 1200

``` r
length(score)
```

    ## [1] 1200

``` r
length(start_date)
```

    ## [1] 1200

``` r
length(anime_title)
```

    ## [1] 1200

We can see that the links information have the same length and the span
information also have the same lengths. However, in order to combine the
two we will need to clean up the link information. Lets look at each of
the collected information below:

``` r
knitr::kable(head(link))
```

| x                                                            |
|:-------------------------------------------------------------|
| <https://myanimelist.net/anime/producer/102/Funimation>      |
| <https://myanimelist.net/anime/producer/102/Funimation/news> |
| <https://myanimelist.net/>                                   |
| <https://myanimelist.net/company>                            |
| <https://myanimelist.net/anime/producer/102/Funimation>      |
| <https://myanimelist.net/anime/producer/102/Funimation/news> |

``` r
knitr::kable(head(link_text))
```

| x          |
|:-----------|
| Details    |
| News       |
| Top        |
| Companies  |
| Funimation |
| More News  |

``` r
knitr::kable(head(members))
```

| x       |
|:--------|
| 1557631 |
| 641079  |
| 103412  |
| 146845  |
| 1789268 |
| 301804  |

``` r
knitr::kable(head(score))
```

| x    |
|:-----|
| 8.76 |
| 8.22 |
| 7.26 |
| 8.14 |
| 8.62 |
| 7.90 |

``` r
knitr::kable(head(start_date))
```

| x        |
|:---------|
| 19980403 |
| 19980401 |
| 20020702 |
| 20040417 |
| 19991020 |
| 20041005 |

``` r
knitr::kable(head(anime_title))
```

| x                      |
|:-----------------------|
| Cowboy Bebop           |
| Trigun                 |
| Witch Hunter Robin     |
| Initial D Fourth Stage |
| One Piece              |
| School Rumble          |

#### Cleaning the link information

Since links needs some work, lets merged them together:

``` r
link_combo <- data.frame(link = link, link_text = link_text, stringsAsFactors = FALSE)
knitr::kable(head(link_combo, n=20))
```

| link                                                                                                 | link\_text                                   |
|:-----------------------------------------------------------------------------------------------------|:---------------------------------------------|
| <https://myanimelist.net/anime/producer/102/Funimation>                                              | Details                                      |
| <https://myanimelist.net/anime/producer/102/Funimation/news>                                         | News                                         |
| <https://myanimelist.net/>                                                                           | Top                                          |
| <https://myanimelist.net/company>                                                                    | Companies                                    |
| <https://myanimelist.net/anime/producer/102/Funimation>                                              | Funimation                                   |
| <https://myanimelist.net/anime/producer/102/Funimation/news>                                         | More News                                    |
| <https://myanimelist.net/news/61353632>                                                              |                                              |
| <https://myanimelist.net/news/61353632>                                                              | Funimation Global Group Acquires Crunchyroll |
| <https://myanimelist.net/profile/ImperfectBlue>                                                      | ImperfectBlue                                |
| <https://myanimelist.net/forum/?topicid=1880964>                                                     | 78 Comments                                  |
| <https://myanimelist.net/news/tag/companies>                                                         | Companies                                    |
| NA                                                                                                   |                                              |
| NA                                                                                                   |                                              |
| NA                                                                                                   |                                              |
| <https://myanimelist.net/anime/1/Cowboy_Bebop>                                                       |                                              |
| <https://myanimelist.net/anime/1/Cowboy_Bebop>                                                       | Cowboy Bebop                                 |
| <https://myanimelist.net/login.php?error=login_required&from=%2Fanime%2Fproducer%2F102%2FFunimation> |                                              |
| <https://myanimelist.net/anime/6/Trigun>                                                             |                                              |
| <https://myanimelist.net/anime/6/Trigun>                                                             | Trigun                                       |
| <https://myanimelist.net/login.php?error=login_required&from=%2Fanime%2Fproducer%2F102%2FFunimation> |                                              |

It looks like the first 15 links aren’t pertinent to what we’re looking
for so lets get rid of those:

``` r
link_combo <- tail(link_combo, n=3599)
knitr::kable(head(link_combo))
```

|     | link                                                                                                 | link\_text   |
|:----|:-----------------------------------------------------------------------------------------------------|:-------------|
| 16  | <https://myanimelist.net/anime/1/Cowboy_Bebop>                                                       | Cowboy Bebop |
| 17  | <https://myanimelist.net/login.php?error=login_required&from=%2Fanime%2Fproducer%2F102%2FFunimation> |              |
| 18  | <https://myanimelist.net/anime/6/Trigun>                                                             |              |
| 19  | <https://myanimelist.net/anime/6/Trigun>                                                             | Trigun       |
| 20  | <https://myanimelist.net/login.php?error=login_required&from=%2Fanime%2Fproducer%2F102%2FFunimation> |              |
| 21  | <https://myanimelist.net/anime/7/Witch_Hunter_Robin>                                                 |              |

Now, lets look at the bottom of the list

``` r
knitr::kable(tail(link_combo))
```

|      | link                                                                                                 | link\_text                                      |
|:-----|:-----------------------------------------------------------------------------------------------------|:------------------------------------------------|
| 3609 | <https://myanimelist.net/anime/49930/Genjitsu_Shugi_Yuusha_no_Oukoku_Saikenki_Part_2>                |                                                 |
| 3610 | <https://myanimelist.net/anime/49930/Genjitsu_Shugi_Yuusha_no_Oukoku_Saikenki_Part_2>                | Genjitsu Shugi Yuusha no Oukoku Saikenki Part 2 |
| 3611 | <https://myanimelist.net/login.php?error=login_required&from=%2Fanime%2Fproducer%2F102%2FFunimation> |                                                 |
| 3612 | <https://myanimelist.net/anime/49969/Tribe_Nine>                                                     |                                                 |
| 3613 | <https://myanimelist.net/anime/49969/Tribe_Nine>                                                     | Tribe Nine                                      |
| 3614 | <https://myanimelist.net/login.php?error=login_required&from=%2Fanime%2Fproducer%2F102%2FFunimation> |                                                 |

The last one can be dropped off and we also want to remove the entries
that don’t have text associated.

``` r
nrow(link_combo)
```

    ## [1] 3599

``` r
link_combo <- head(link_combo, n=3598)
lapply(link_combo,function(x) { length(which(is.na(x)))})
```

    ## $link
    ## [1] 0
    ## 
    ## $link_text
    ## [1] 0

So none of the columns are null by looking at the table, however, many
of them are filled with empty strings.

We can run the following script to remove any rows that have an empty
string

``` r
link_combo <- link_combo[!apply(link_combo == "", 1, any), ] 
nrow(link_combo)
```

    ## [1] 1200

``` r
knitr::kable(head(link_combo))
```

|     | link                                                      | link\_text             |
|:----|:----------------------------------------------------------|:-----------------------|
| 16  | <https://myanimelist.net/anime/1/Cowboy_Bebop>            | Cowboy Bebop           |
| 19  | <https://myanimelist.net/anime/6/Trigun>                  | Trigun                 |
| 22  | <https://myanimelist.net/anime/7/Witch_Hunter_Robin>      | Witch Hunter Robin     |
| 25  | <https://myanimelist.net/anime/18/Initial_D_Fourth_Stage> | Initial D Fourth Stage |
| 28  | <https://myanimelist.net/anime/21/One_Piece>              | One Piece              |
| 31  | <https://myanimelist.net/anime/24/School_Rumble>          | School Rumble          |

**Note**: using the ‘any’ function allows removal of any row that has an
empty string. If I wanted to remove rows which had empty strings in all
columns, I would simply change ‘any’ to ‘all’.

Now we have all dataframes and vectors of the same length and we can
combine them into one dataframe.

Before I do that, I want to extract the mal\_id from the link. This is
because I have another table with this ID that I would like to join at
some point.

To do that, I need to extract it from the link. There are a couple of
ways to do this:

-   I can use the dplyr package to separate the link column based on a
    delimiter ‘/’ then pick the correct entry as my mal\_id
-   I can use a general expression to extract the item and add it as a
    new column.

Lets do the latter since general expressions are vital for string
operations.

``` r
#For details on how to create a regex for extracting a string between two strings: https://regexland.com/all-between-specified-characters/
link_combo$mal_id <- str_extract(link_combo$link, "(?<=anime/).*(?=/)")
knitr::kable(head(link_combo))
```

|     | link                                                      | link\_text             | mal\_id |
|:----|:----------------------------------------------------------|:-----------------------|:--------|
| 16  | <https://myanimelist.net/anime/1/Cowboy_Bebop>            | Cowboy Bebop           | 1       |
| 19  | <https://myanimelist.net/anime/6/Trigun>                  | Trigun                 | 6       |
| 22  | <https://myanimelist.net/anime/7/Witch_Hunter_Robin>      | Witch Hunter Robin     | 7       |
| 25  | <https://myanimelist.net/anime/18/Initial_D_Fourth_Stage> | Initial D Fourth Stage | 18      |
| 28  | <https://myanimelist.net/anime/21/One_Piece>              | One Piece              | 21      |
| 31  | <https://myanimelist.net/anime/24/School_Rumble>          | School Rumble          | 24      |

#### Combining all the elements

Ok, now lets put them all together

``` r
funimation_table <- data.frame(mal_id = link_combo$mal_id, anime_title = link_combo$link_text, score = score, members = members, start_date = start_date, url = link_combo$link, url_name = link_combo$link_text, stringsAsFactors = FALSE)
identical(funimation_table[['anime_title']],funimation_table[['url_name']])
```

    ## [1] TRUE

Checking to ensure that the anime\_title and the url\_name are identical
is a way to ensure that the order of the two datasets are aligned.

Lets see what the table looks like now:

``` r
knitr::kable(head(funimation_table))
```

| mal\_id | anime\_title           | score | members | start\_date | url                                                       | url\_name              |
|:--------|:-----------------------|:------|:--------|:------------|:----------------------------------------------------------|:-----------------------|
| 1       | Cowboy Bebop           | 8.76  | 1557631 | 19980403    | <https://myanimelist.net/anime/1/Cowboy_Bebop>            | Cowboy Bebop           |
| 6       | Trigun                 | 8.22  | 641079  | 19980401    | <https://myanimelist.net/anime/6/Trigun>                  | Trigun                 |
| 7       | Witch Hunter Robin     | 7.26  | 103412  | 20020702    | <https://myanimelist.net/anime/7/Witch_Hunter_Robin>      | Witch Hunter Robin     |
| 18      | Initial D Fourth Stage | 8.14  | 146845  | 20040417    | <https://myanimelist.net/anime/18/Initial_D_Fourth_Stage> | Initial D Fourth Stage |
| 21      | One Piece              | 8.62  | 1789268 | 19991020    | <https://myanimelist.net/anime/21/One_Piece>              | One Piece              |
| 24      | School Rumble          | 7.90  | 301804  | 20041005    | <https://myanimelist.net/anime/24/School_Rumble>          | School Rumble          |

We have a redundant title column that can be removed, then we can export
it.

``` r
funimation_table <- funimation_table[, 1:(length(funimation_table)-1)]
knitr::kable(head(funimation_table))
```

| mal\_id | anime\_title           | score | members | start\_date | url                                                       |
|:--------|:-----------------------|:------|:--------|:------------|:----------------------------------------------------------|
| 1       | Cowboy Bebop           | 8.76  | 1557631 | 19980403    | <https://myanimelist.net/anime/1/Cowboy_Bebop>            |
| 6       | Trigun                 | 8.22  | 641079  | 19980401    | <https://myanimelist.net/anime/6/Trigun>                  |
| 7       | Witch Hunter Robin     | 7.26  | 103412  | 20020702    | <https://myanimelist.net/anime/7/Witch_Hunter_Robin>      |
| 18      | Initial D Fourth Stage | 8.14  | 146845  | 20040417    | <https://myanimelist.net/anime/18/Initial_D_Fourth_Stage> |
| 21      | One Piece              | 8.62  | 1789268 | 19991020    | <https://myanimelist.net/anime/21/One_Piece>              |
| 24      | School Rumble          | 7.90  | 301804  | 20041005    | <https://myanimelist.net/anime/24/School_Rumble>          |

``` r
write.csv(funimation_table, 'funimationtitles.csv', row.names = FALSE) 
```
