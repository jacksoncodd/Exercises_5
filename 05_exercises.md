---
title: 'Weekly Exercises #5'
author: "Jackson Codd"
output: 
  html_document:
    keep_md: TRUE
    toc: TRUE
    toc_float: TRUE
    df_print: paged
    code_download: true
---





```r
library(tidyverse)     # for data cleaning and plotting
library(googlesheets4) # for reading googlesheet data
library(lubridate)     # for date manipulation
library(openintro)     # for the abbr2state() function
library(palmerpenguins)# for Palmer penguin data
library(maps)          # for map data
library(ggmap)         # for mapping points on maps
library(gplots)        # for col2hex() function
library(RColorBrewer)  # for color palettes
library(sf)            # for working with spatial data
library(leaflet)       # for highly customizable mapping
library(ggthemes)      # for more themes (including theme_map())
library(plotly)        # for the ggplotly() - basic interactivity
library(gganimate)     # for adding animation layers to ggplots
library(transformr)    # for "tweening" (gganimate)
library(shiny)         # for creating interactive apps
gs4_deauth()           # To not have to authorize each time you knit.
theme_set(theme_minimal())
```


```r
# SNCF Train data
small_trains <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-02-26/small_trains.csv") 

# Lisa's garden data
garden_harvest <- read_sheet("https://docs.google.com/spreadsheets/d/1DekSazCzKqPS2jnGhKue7tLxRU3GVL1oxi-4bEM5IWw/edit?usp=sharing") %>% 
  mutate(date = ymd(date))

# Lisa's Mallorca cycling data
mallorca_bike_day7 <- read_csv("https://www.dropbox.com/s/zc6jan4ltmjtvy0/mallorca_bike_day7.csv?dl=1") %>% 
  select(1:4, speed)

# Heather Lendway's Ironman 70.3 Pan Am championships Panama data
panama_swim <- read_csv("https://raw.githubusercontent.com/llendway/gps-data/master/data/panama_swim_20160131.csv")

panama_bike <- read_csv("https://raw.githubusercontent.com/llendway/gps-data/master/data/panama_bike_20160131.csv")

panama_run <- read_csv("https://raw.githubusercontent.com/llendway/gps-data/master/data/panama_run_20160131.csv")

#COVID-19 data from the New York Times
covid19 <- read_csv("https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv")
```

## Put your homework on GitHub!

Go [here](https://github.com/llendway/github_for_collaboration/blob/master/github_for_collaboration.md) or to previous homework to remind yourself how to get set up. 

Once your repository is created, you should always open your **project** rather than just opening an .Rmd file. You can do that by either clicking on the .Rproj file in your repository folder on your computer. Or, by going to the upper right hand corner in R Studio and clicking the arrow next to where it says Project: (None). You should see your project come up in that list if you've used it recently. You could also go to File --> Open Project and navigate to your .Rproj file. 

## Instructions

* Put your name at the top of the document. 

* **For ALL graphs, you should include appropriate labels.** 

* Feel free to change the default theme, which I currently have set to `theme_minimal()`. 

* Use good coding practice. Read the short sections on good code with [pipes](https://style.tidyverse.org/pipes.html) and [ggplot2](https://style.tidyverse.org/ggplot2.html). **This is part of your grade!**

* **NEW!!** With animated graphs, add `eval=FALSE` to the code chunk that creates the animation and saves it using `anim_save()`. Add another code chunk to reread the gif back into the file. See the [tutorial](https://animation-and-interactivity-in-r.netlify.app/) for help. 

* When you are finished with ALL the exercises, uncomment the options at the top so your document looks nicer. Don't do it before then, or else you might miss some important warnings and messages.

## Warm-up exercises from tutorial

  1. Choose 2 graphs you have created for ANY assignment in this class and add interactivity using the `ggplotly()` function.
  
  

```r
library(babynames)   
data_site <- 
  "https://www.macalester.edu/~dshuman1/data/112/2014-Q4-Trips-History-Data.rds" 
Trips <- readRDS(gzcon(url(data_site)))
Stations<-read_csv("http://www.macalester.edu/~dshuman1/data/112/DC-Stations.csv")
```
  

```r
babynames %>%
  group_by(year, sex) %>%
  summarise(obs = n(), num = n_distinct(name)) %>%
  ggplot(aes(year, num)) +
  geom_line(aes(color=sex)) +
  labs(title = "Number of Used Names Since 1880", 
       x = "Year", 
       y = "Names") +
  transition_reveal(year) -> name_by_year
  
animate(name_by_year, nframes = 200, duration = 10, renderer = gifski_renderer())
anim_save("name_by_year.gif")
```
![](name_by_year.gif)<!-- -->


```r
Trips %>%
  mutate(wday = wday(sdate, label = TRUE)) %>%
  mutate(hour = hour(sdate)) %>%
  mutate(minute = minute(sdate)) %>%
  mutate(time = hour+(minute/60)) %>%
  ggplot(aes(time, fill= client, alpha = .5)) +
  geom_density(color = NA, position = position_stack()) +
  labs(title = "Bike Rentals Over Day", 
       subtitle = "{closest_state}", 
       x = "Time", 
       y = "Density") +
  transition_states(wday) -> bikers_by_wday

animate(bikers_by_wday, nframes = 200, duration = 10, renderer = gifski_renderer())
anim_save("bikers_by_wday.gif")
```

![](bikers_by_wday.gif)<!-- -->


  
  2. Use animation to tell an interesting story with the `small_trains` dataset that contains data from the SNCF (National Society of French Railways). These are Tidy Tuesday data! Read more about it [here](https://github.com/rfordatascience/tidytuesday/tree/master/data/2019/2019-02-26).


```r
small_trains %>%
  filter(service == "International") %>%
  group_by(departure_station, month) %>%
  mutate(avg_delay_yearly = avg_delay_all_arriving-avg_delay_all_departing) %>%
  summarise(avg_delay = mean(avg_delay_yearly)) %>%
  ggplot(aes(x = month, y = avg_delay)) +
  geom_point() +
  labs(title = "Average Delay Per Month on International Trips", 
       subtitle = "Origin: {closest_state}",
       x = "Month",
       y = "Average Delay (Minutes)") +
  transition_states(departure_station) -> delay_station

animate(delay_station, nframes = 200, duration = 15, renderer = gifski_renderer())
anim_save("delay_station.gif")
```

![](delay_station.gif)<!-- -->

## Garden data

  3. In this exercise, you will create a stacked area plot that reveals itself over time (see the `geom_area()` examples [here](https://ggplot2.tidyverse.org/reference/position_stack.html)). You will look at cumulative harvest of tomato varieties over time. You should do the following:
  * From the `garden_harvest` data, filter the data to the tomatoes and find the *daily* harvest in pounds for each variety.  
  * Then, for each variety, find the cumulative harvest in pounds.  
  * Use the data you just made to create a static cumulative harvest area plot, with the areas filled with different colors for each vegetable and arranged (HINT: `fct_reorder()`) from most to least harvested (most on the bottom).  
  * Add animation to reveal the plot over date. 


```r
garden_harvest %>%
  filter(vegetable == "tomatoes") %>%
  complete(variety, date = seq.Date(min(date), max(date), by = "day")) %>%
  mutate(weight = replace_na(weight, 0)) %>%
  group_by(variety, date) %>%
  summarise(daily_harvest_lbs = sum(weight)*.00220462) %>%
  mutate(cum_harvest = cumsum(daily_harvest_lbs)) %>%
  ungroup() %>%
  group_by(variety) %>%
  mutate(tot_harvest = sum(daily_harvest_lbs)) %>%
  mutate(variety = fct_reorder(variety, tot_harvest, min)) %>%
  ggplot(aes(date, cum_harvest)) +
  geom_area(aes(fill = variety)) +
  labs(title = "Cumulative Harvest by Date", x = "Date", y = "Harvest (lbs)") +
  transition_reveal(date) -> cum_harvest_date

animate(cum_harvest_date, nframes = 100, duration = 8, renderer = gifski_renderer())
anim_save("harvest_date.gif")
```
![](harvest_date.gif)<!-- -->

## Maps, animation, and movement!

  4. Map my `mallorca_bike_day7` bike ride using animation! 
  Requirements:
  * Plot on a map using `ggmap`.  
  * Show "current" location with a red point. 
  * Show path up until the current point.  
  * Color the path according to elevation.  
  * Show the time in the subtitle.  
  * CHALLENGE: use the `ggimage` package and `geom_image` to add a bike image instead of a red point. You can use [this](https://raw.githubusercontent.com/llendway/animation_and_interactivity/master/bike.png) image. See [here](https://goodekat.github.io/presentations/2019-isugg-gganimate-spooky/slides.html#35) for an example. 
  * Add something of your own! And comment on if you prefer this to the static map and why or why not.
  

```r
mallorca <- get_stamenmap(
    bbox = c(left = 2.3410, bottom = 39.5354, right = 2.7, top = 39.7402), 
    maptype = "terrain",
    zoom = 9)

ggmap(mallorca) +
  geom_point(data = mallorca_bike_day7,
             aes(lon, lat, size = speed), 
             color = "red") +
  theme_map() +
  theme(legend.background = element_blank()) +
  transition_time(time) +
  geom_line(data = mallorca_bike_day7,
            aes(lon, lat, color = ele)) +
  theme_map() +
  theme(legend.background = element_blank()) +
  labs(title = "Mallorca Bike Ride", subtitle = "Time: {frame_along}") +
  transition_reveal(time) -> mallorca_path

animate(mallorca_path, nframes = 200, duration = 15, renderer = gifski_renderer())
anim_save("mallorca_path.gif")
```
![](mallorca_path.gif)<!-- -->

I definitely prefer this to the static map because it gives much more information and feels more interactive.

  5. In this exercise, you get to meet my sister, Heather! She is a proud Mac grad, currently works as a Data Scientist at 3M where she uses R everyday, and for a few years (while still holding a full-time job) she was a pro triathlete. You are going to map one of her races. The data from each discipline of the Ironman 70.3 Pan Am championships, Panama is in a separate file - `panama_swim`, `panama_bike`, and `panama_run`. Create a similar map to the one you created with my cycling data. You will need to make some small changes: 1. combine the files (HINT: `bind_rows()`, 2. make the leading dot a different color depending on the event (for an extra challenge, make it a different image using `geom_image()!), 3. CHALLENGE (optional): color by speed, which you will need to compute on your own from the data. You can read Heather's race report [here](https://heatherlendway.com/2016/02/10/ironman-70-3-pan-american-championships-panama-race-report/). She is also in the Macalester Athletics [Hall of Fame](https://athletics.macalester.edu/honors/hall-of-fame/heather-lendway/184) and still has records at the pool. 
  

```r
panama_swim %>%
  bind_rows(panama_bike) %>%
  bind_rows(panama_run) ->panama_tri
  
panama <- get_stamenmap(
    bbox = c(left = -79.650, bottom = 8.9, right = -79.460, top = 8.996), 
    maptype = "terrain",
    zoom = 11)

ggmap(panama) +
  geom_point(data = panama_tri,
             aes(lon, lat, color = event),
             size = 2) +
  theme_map() +
  transition_time(time) +
  geom_line(data = panama_tri,
            aes(lon, lat, color = event)) +
  theme_map() +
  theme(legend.background = element_blank()) +
  labs(title = "Panama Triathalon Path", subtitle = "Time: {frame_along}") +
  transition_reveal(time) -> panama_triathlon

animate(panama_triathlon, nframes = 200, duration = 25, renderer = gifski_renderer())
anim_save("panama_triathlon.gif")
```

![](panama_triathlon.gif)<!-- -->
  
## COVID-19 data

  6. In this exercise, you are going to replicate many of the features in [this](https://aatishb.com/covidtrends/?region=US) visualization by Aitish Bhatia but include all US states. Requirements:
 * Create a new variable that computes the number of new cases in the past week (HINT: use the `lag()` function you've used in a previous set of exercises). Replace missing values with 0's using `replace_na()`.  
  * Filter the data to omit rows where the cumulative case counts are less than 20.  
  * Create a static plot with cumulative cases on the x-axis and new cases in the past 7 days on the x-axis. Connect the points for each state over time. HINTS: use `geom_path()` and add a `group` aesthetic.  Put the x and y axis on the log scale and make the tick labels look nice - `scales::comma` is one option. This plot will look pretty ugly as is.
  * Animate the plot to reveal the pattern by date. Display the date as the subtitle. Add a leading point to each state's line (`geom_point()`) and add the state name as a label (`geom_text()` - you should look at the `check_overlap` argument).  
  * Use the `animate()` function to have 200 frames in your animation and make it 30 seconds long. 
  * Comment on what you observe.
  

```r
covid19 %>%
  group_by(state) %>%
  mutate(new = lag(cases, n = 7)) %>%
  replace_na(list(new = 0)) %>%
  filter(cases >= 20) %>%
  ggplot(aes(x = cases, y = new,
             group = state)) +
  geom_point() +
  geom_text(aes(label = state), check_overlap = TRUE) +
  geom_path() +
  scale_y_log10(label = scales::comma) +
  scale_x_log10(label = scales::comma) +
  labs(title = "COVID Cases by State", 
       subtitle = "Date: {frame_along}", 
       x = "Total Cases", 
       y = "Weekly New Cases") +
  transition_reveal(date) -> state_cases
  

animate(state_cases, nframes = 200, duration = 30, renderer = gifski_renderer())
anim_save("state_cases.gif")
```

![](state_cases.gif)<!-- -->

An interesting aspect of the dataset is that even though the states started below the line that can be observed, almost all of them are now along it, though at different places.

  
  7. In this exercise you will animate a map of the US, showing how cumulative COVID-19 cases per 10,000 residents has changed over time. This is similar to exercises 11 & 12 from the previous exercises, with the added animation! So, in the end, you should have something like the static map you made there, but animated over all the days. Put date in the subtitle. Comment on what you see.
  

```r
census_pop_est_2018 <- read_csv("https://www.dropbox.com/s/6txwv3b4ng7pepe/us_census_2018_state_pop_est.csv?dl=1") %>% 
  separate(state, into = c("dot","state"), extra = "merge") %>% 
  select(-dot) %>% 
  mutate(state = str_to_lower(state))
```

```r
states_map <- map_data("state")
covid19 %>% 
  group_by(state) %>%
  #complete(state, date = seq.Date(min(date), max(date), by = "day")) %>%
  #mutate(cases = replace_na(cases, 0)) %>%
  mutate(state = str_to_lower(state),
         weekday = wday(date, label = TRUE)) %>%
  filter(weekday == "Fri") %>%
  left_join(census_pop_est_2018,
            by = "state") %>%
  mutate(cases_per_10000 = cases/(est_pop_2018/10000)) %>%
  ggplot() +
  geom_map(map = states_map,
           aes(map_id = state,
               fill = cases_per_10000,
               group = date)) +
  expand_limits(x = states_map$long, y = states_map$lat) + 
  labs(title = "COVID Cases by State per 10,000 Population", 
       subtitle = "{closest_state}") +
  scale_fill_viridis_c(option = "viridis") +
  theme_map() +
  theme(legend.background = element_blank()) +
  transition_states(date) -> state_10000

animate(state_10000, nframes = 200, duration = 30, renderer = gifski_renderer())
anim_save("state_10000.gif")
```

![](state_10000.gif)<!-- -->

Though states on the west coast and northeast have much larger numbers of infected early on, by the end they have slowed down the spread and southern states have pulled ahead.


## Your first `shiny` app

  8. This app will also use the COVID data. Make sure you load that data and all the libraries you need in the `app.R` file you create. Below, you will post a link to the app that you publish on shinyapps.io. You will create an app to compare states' cumulative number of COVID cases over time. The x-axis will be number of days since 20+ cases and the y-axis will be cumulative cases on the log scale (`scale_y_log10()`). We use number of days since 20+ cases on the x-axis so we can make better comparisons of the curve trajectories. You will have an input box where the user can choose which states to compare (`selectInput()`) and have a submit button to click once the user has chosen all states they're interested in comparing. The graph should display a different line for each state, with labels either on the graph or in a legend. Color can be used if needed. 
  
## GitHub link

  9. Below, provide a link to your GitHub page with this set of Weekly Exercises. Specifically, if the name of the file is 05_exercises.Rmd, provide a link to the 05_exercises.md file, which is the one that will be most readable on GitHub. If that file isn't very readable, then provide a link to your main GitHub page.


**DID YOU REMEMBER TO UNCOMMENT THE OPTIONS AT THE TOP?**
