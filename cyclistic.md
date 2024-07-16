![](cyclistic_files/figure-markdown_strict/divvyfile-1-900x600.jpg)

# Cyclistic Bike-Share Marketing Analysis by R

**Author:** Adela Xu  
**Date:** 2024-07-12

## **Introduction**

This case study is a capstone project for the Google Data Analyst
Professional Certificate. In this project, I employed analytical and
data visualization skills to address a marketing problem for Cyclistic,
a fictional bike share company in Chicago.

### **Scenario**

I am a junior data analyst on the marketing analytics team at Cyclistic,
a bike-share company in Chicago. The director of marketing believes that
the company’s future success depends on increasing the number of annual
subscribers. To achieve this, our team aims to understand how casual
customers and annual subscribers use Cyclistic bikes differently. With
these insights, we will design a new marketing strategy to convert
casual customers into annual subscribers.

### **About the company**

In 2016, Cyclistic launched a successful bike-share offering. Since
then, the program has grown to a fleet of 5,824 bicycles that are
geotracked and locked into a network of 692 stations across Chicago. The
bikes can be unlocked form one station and returned to any other station
in the system anytime.

Until now, Cyclistic’s marketing strategy relied on building general
awareness and appealing to broad consumer segments. Cyclistic’s finance
analysts have concluded that annual members are much more profitable
than casual riders. Thus, the marketing director (my manager) has set a
clear goal: Design marketing strategies aimed at converting casual
customers into annual subscribers. In order to do that, the team needs
to better understand how annual subscribers and casual customers differ,
why casual customers would buy a subscription, and how digital media
could affect their marketing tactics. Our team are interested in
analyzing the Cyclistic historical bike trip data to identify trends.

## **Ask Phase**

### **Business Tasks**

As a data analyst on the marketing team, my job is to answer the
question: **How do annual subscribers and casual customers use Cyclistic
bikes differently?** I will present a detailed analysis supported by
data visualizations, share the key findings, and provide recommendations
accordingly.

### **Questions to be answered**

-   What are the proportions of subscribers and casual customers?
-   How do trip durations differ for subscribers and casual customers?
-   How do trip frequencies differ between weekdays and weekends for
    both groups?
-   Are there noticeable differences in trip purposes between members
    and casual riders?
-   What are the peak hours during weekdays and weekends for both
    subscribers and casual customers?
-   How does bike usage vary across different seasons for both
    subscribers and casual customers?
-   Which stations are more popular among subscribers and casual
    customers?
-   How do the demographics (e.g., birth year, gender) of subscribers
    compare to those of casual customers?

## **Prepare Phase**

I analyzed Cyclistic’s historical trip data from all four quarters of
2019 to identify trends. The data is downloaded from
[here](https://divvy-tripdata.s3.amazonaws.com/index.html) and is
provided by Motivate International Inc. under this
[license](https://divvybikes.com/data-license-agreement). Each of the
four files is contains twelve columns of data for all rides that
occurred between January 2019 and December 2019. The credibility of the
data source is confirmed.

## **Process Phase**

    # Load required packages
    library(tidyverse)
    library(dplyr)
    library(ggplot2)
    library(lubridate)
    library(rmarkdown)
    knitr::opts_chunk$set(echo = TRUE)

    # Upload the four CSV files for the trip data of 2019
    Q1_2019 <- read.csv("Divvy_Trips_2019_Q1.csv")
    Q2_2019 <- read.csv("Divvy_Trips_2019_Q2.csv")
    Q3_2019 <- read.csv("Divvy_Trips_2019_Q3.csv")
    Q4_2019 <- read.csv("Divvy_Trips_2019_Q4.csv")

    # Review and compare column names of each file
    # While the names don't have to be in the same order, we want them to match perfectly before we use a command to combine them into one file
    colnames(Q1_2019)

    ##  [1] "trip_id"           "start_time"        "end_time"          "bikeid"           
    ##  [5] "tripduration"      "from_station_id"   "from_station_name" "to_station_id"    
    ##  [9] "to_station_name"   "usertype"          "gender"            "birthyear"

    colnames(Q2_2019)

    ##  [1] "X01...Rental.Details.Rental.ID"                   
    ##  [2] "X01...Rental.Details.Local.Start.Time"            
    ##  [3] "X01...Rental.Details.Local.End.Time"              
    ##  [4] "X01...Rental.Details.Bike.ID"                     
    ##  [5] "X01...Rental.Details.Duration.In.Seconds.Uncapped"
    ##  [6] "X03...Rental.Start.Station.ID"                    
    ##  [7] "X03...Rental.Start.Station.Name"                  
    ##  [8] "X02...Rental.End.Station.ID"                      
    ##  [9] "X02...Rental.End.Station.Name"                    
    ## [10] "User.Type"                                        
    ## [11] "Member.Gender"                                    
    ## [12] "X05...Member.Details.Member.Birthday.Year"

    colnames(Q3_2019)

    ##  [1] "trip_id"           "start_time"        "end_time"          "bikeid"           
    ##  [5] "tripduration"      "from_station_id"   "from_station_name" "to_station_id"    
    ##  [9] "to_station_name"   "usertype"          "gender"            "birthyear"

    colnames(Q4_2019)

    ##  [1] "trip_id"           "start_time"        "end_time"          "bikeid"           
    ##  [5] "tripduration"      "from_station_id"   "from_station_name" "to_station_id"    
    ##  [9] "to_station_name"   "usertype"          "gender"            "birthyear"

    # Q2 dataset's column names are different from the others. Rename columns to make Q2 data consistent with the other 3 quarters'.
    Q2_2019 <- rename(Q2_2019 
                      ,trip_id = X01...Rental.Details.Rental.ID
                      ,start_time = X01...Rental.Details.Local.Start.Time
                      ,end_time = X01...Rental.Details.Local.End.Time
                      ,bikeid = X01...Rental.Details.Bike.ID
                      ,tripduration = X01...Rental.Details.Duration.In.Seconds.Uncapped
                      ,from_station_id = X03...Rental.Start.Station.ID
                      ,from_station_name = X03...Rental.Start.Station.Name
                      ,to_station_id = X02...Rental.End.Station.ID
                      ,to_station_name = X02...Rental.End.Station.Name
                      ,usertype = User.Type
                      ,gender = Member.Gender
                      ,birthyear = X05...Member.Details.Member.Birthday.Year
                      )

    # Inspect the dataframes and look for incongruences. 
    str(Q1_2019)

    ## 'data.frame':    365069 obs. of  12 variables:
    ##  $ trip_id          : int  21742443 21742444 21742445 21742446 21742447 21742448 21742449 21742450 21742451 21742452 ...
    ##  $ start_time       : chr  "2019-01-01 00:04:37" "2019-01-01 00:08:13" "2019-01-01 00:13:23" "2019-01-01 00:13:45" ...
    ##  $ end_time         : chr  "2019-01-01 00:11:07" "2019-01-01 00:15:34" "2019-01-01 00:27:12" "2019-01-01 00:43:28" ...
    ##  $ bikeid           : int  2167 4386 1524 252 1170 2437 2708 2796 6205 3939 ...
    ##  $ tripduration     : chr  "390.0" "441.0" "829.0" "1,783.0" ...
    ##  $ from_station_id  : int  199 44 15 123 173 98 98 211 150 268 ...
    ##  $ from_station_name: chr  "Wabash Ave & Grand Ave" "State St & Randolph St" "Racine Ave & 18th St" "California Ave & Milwaukee Ave" ...
    ##  $ to_station_id    : int  84 624 644 176 35 49 49 142 148 141 ...
    ##  $ to_station_name  : chr  "Milwaukee Ave & Grand Ave" "Dearborn St & Van Buren St (*)" "Western Ave & Fillmore St (*)" "Clark St & Elm St" ...
    ##  $ usertype         : chr  "Subscriber" "Subscriber" "Subscriber" "Subscriber" ...
    ##  $ gender           : chr  "Male" "Female" "Female" "Male" ...
    ##  $ birthyear        : int  1989 1990 1994 1993 1994 1983 1984 1990 1995 1996 ...

    str(Q2_2019)

    ## 'data.frame':    1108163 obs. of  12 variables:
    ##  $ trip_id          : int  22178529 22178530 22178531 22178532 22178533 22178534 22178535 22178536 22178537 22178538 ...
    ##  $ start_time       : chr  "2019-04-01 00:02:22" "2019-04-01 00:03:02" "2019-04-01 00:11:07" "2019-04-01 00:13:01" ...
    ##  $ end_time         : chr  "2019-04-01 00:09:48" "2019-04-01 00:20:30" "2019-04-01 00:15:19" "2019-04-01 00:18:58" ...
    ##  $ bikeid           : int  6251 6226 5649 4151 3270 3123 6418 4513 3280 5534 ...
    ##  $ tripduration     : chr  "446.0" "1,048.0" "252.0" "357.0" ...
    ##  $ from_station_id  : int  81 317 283 26 202 420 503 260 211 211 ...
    ##  $ from_station_name: chr  "Daley Center Plaza" "Wood St & Taylor St" "LaSalle St & Jackson Blvd" "McClurg Ct & Illinois St" ...
    ##  $ to_station_id    : int  56 59 174 133 129 426 500 499 211 211 ...
    ##  $ to_station_name  : chr  "Desplaines St & Kinzie St" "Wabash Ave & Roosevelt Rd" "Canal St & Madison St" "Kingsbury St & Kinzie St" ...
    ##  $ usertype         : chr  "Subscriber" "Subscriber" "Subscriber" "Subscriber" ...
    ##  $ gender           : chr  "Male" "Female" "Male" "Male" ...
    ##  $ birthyear        : int  1975 1984 1990 1993 1992 1999 1969 1991 NA NA ...

    str(Q3_2019)

    ## 'data.frame':    1640718 obs. of  12 variables:
    ##  $ trip_id          : int  23479388 23479389 23479390 23479391 23479392 23479393 23479394 23479395 23479396 23479397 ...
    ##  $ start_time       : chr  "2019-07-01 00:00:27" "2019-07-01 00:01:16" "2019-07-01 00:01:48" "2019-07-01 00:02:07" ...
    ##  $ end_time         : chr  "2019-07-01 00:20:41" "2019-07-01 00:18:44" "2019-07-01 00:27:42" "2019-07-01 00:27:10" ...
    ##  $ bikeid           : int  3591 5353 6180 5540 6014 4941 3770 5442 2957 6091 ...
    ##  $ tripduration     : chr  "1,214.0" "1,048.0" "1,554.0" "1,503.0" ...
    ##  $ from_station_id  : int  117 381 313 313 168 300 168 313 43 43 ...
    ##  $ from_station_name: chr  "Wilton Ave & Belmont Ave" "Western Ave & Monroe St" "Lakeview Ave & Fullerton Pkwy" "Lakeview Ave & Fullerton Pkwy" ...
    ##  $ to_station_id    : int  497 203 144 144 62 232 62 144 195 195 ...
    ##  $ to_station_name  : chr  "Kimball Ave & Belmont Ave" "Western Ave & 21st St" "Larrabee St & Webster Ave" "Larrabee St & Webster Ave" ...
    ##  $ usertype         : chr  "Subscriber" "Customer" "Customer" "Customer" ...
    ##  $ gender           : chr  "Male" "" "" "" ...
    ##  $ birthyear        : int  1992 NA NA NA NA 1990 NA NA NA NA ...

    str(Q4_2019)

    ## 'data.frame':    704054 obs. of  12 variables:
    ##  $ trip_id          : int  25223640 25223641 25223642 25223643 25223644 25223645 25223646 25223647 25223648 25223649 ...
    ##  $ start_time       : chr  "2019-10-01 00:01:39" "2019-10-01 00:02:16" "2019-10-01 00:04:32" "2019-10-01 00:04:32" ...
    ##  $ end_time         : chr  "2019-10-01 00:17:20" "2019-10-01 00:06:34" "2019-10-01 00:18:43" "2019-10-01 00:43:43" ...
    ##  $ bikeid           : int  2215 6328 3003 3275 5294 1891 1061 1274 6011 2957 ...
    ##  $ tripduration     : chr  "940.0" "258.0" "850.0" "2,350.0" ...
    ##  $ from_station_id  : int  20 19 84 313 210 156 84 156 156 336 ...
    ##  $ from_station_name: chr  "Sheffield Ave & Kingsbury St" "Throop (Loomis) St & Taylor St" "Milwaukee Ave & Grand Ave" "Lakeview Ave & Fullerton Pkwy" ...
    ##  $ to_station_id    : int  309 241 199 290 382 226 142 463 463 336 ...
    ##  $ to_station_name  : chr  "Leavitt St & Armitage Ave" "Morgan St & Polk St" "Wabash Ave & Grand Ave" "Kedzie Ave & Palmer Ct" ...
    ##  $ usertype         : chr  "Subscriber" "Subscriber" "Subscriber" "Subscriber" ...
    ##  $ gender           : chr  "Male" "Male" "Female" "Male" ...
    ##  $ birthyear        : int  1987 1998 1991 1990 1987 1994 1991 1995 1993 NA ...

    # The data types are consistent across the four datasets for all columns, good to go for combining!

    # Combine the four datasets into one
    all_trips <- bind_rows(Q1_2019, Q2_2019, Q3_2019, Q4_2019)

    # Remove any duplicate trips
    all_trips <- all_trips %>% distinct()

    # Check for missing values in each column
    missing_values <- colSums(is.na(all_trips) | all_trips == "")
    # Print the number of missing values in each column
    print(missing_values)

    ##           trip_id        start_time          end_time            bikeid      tripduration 
    ##                 0                 0                 0                 0                 0 
    ##   from_station_id from_station_name     to_station_id   to_station_name          usertype 
    ##                 0                 0                 0                 0                 0 
    ##            gender         birthyear 
    ##            559206            538751

    # Only gender and birthyear columns have missing values
    # 559206 rows with missing gender, percentage 14.6%
    # 537801 rows with missing birthyear, percentage 14.1%
    # Due to the high percentage of missing values in these two columns, we will retain them for further analysis that does not depend on these columns.

    # check "all_trips" structure
    str(all_trips)

    ## 'data.frame':    3818004 obs. of  12 variables:
    ##  $ trip_id          : int  21742443 21742444 21742445 21742446 21742447 21742448 21742449 21742450 21742451 21742452 ...
    ##  $ start_time       : chr  "2019-01-01 00:04:37" "2019-01-01 00:08:13" "2019-01-01 00:13:23" "2019-01-01 00:13:45" ...
    ##  $ end_time         : chr  "2019-01-01 00:11:07" "2019-01-01 00:15:34" "2019-01-01 00:27:12" "2019-01-01 00:43:28" ...
    ##  $ bikeid           : int  2167 4386 1524 252 1170 2437 2708 2796 6205 3939 ...
    ##  $ tripduration     : chr  "390.0" "441.0" "829.0" "1,783.0" ...
    ##  $ from_station_id  : int  199 44 15 123 173 98 98 211 150 268 ...
    ##  $ from_station_name: chr  "Wabash Ave & Grand Ave" "State St & Randolph St" "Racine Ave & 18th St" "California Ave & Milwaukee Ave" ...
    ##  $ to_station_id    : int  84 624 644 176 35 49 49 142 148 141 ...
    ##  $ to_station_name  : chr  "Milwaukee Ave & Grand Ave" "Dearborn St & Van Buren St (*)" "Western Ave & Fillmore St (*)" "Clark St & Elm St" ...
    ##  $ usertype         : chr  "Subscriber" "Subscriber" "Subscriber" "Subscriber" ...
    ##  $ gender           : chr  "Male" "Female" "Female" "Male" ...
    ##  $ birthyear        : int  1989 1990 1994 1993 1994 1983 1984 1990 1995 1996 ...

    # Convert start_time and end_time to Date-Time
    all_trips <- all_trips %>%
      mutate(start_time = ymd_hms(start_time),
             end_time = ymd_hms(end_time))

    # Convert tripduration to Numeric
    all_trips <- all_trips %>%
      mutate(tripduration = as.numeric(gsub(",", "", tripduration)))

    # Convert tripduration from seconds to minutes
    all_trips <- all_trips %>%
      mutate(tripduration_minutes = tripduration / 60)

    # verify the str again
    str(all_trips)

    ## 'data.frame':    3818004 obs. of  13 variables:
    ##  $ trip_id             : int  21742443 21742444 21742445 21742446 21742447 21742448 21742449 21742450 21742451 21742452 ...
    ##  $ start_time          : POSIXct, format: "2019-01-01 00:04:37" "2019-01-01 00:08:13" "2019-01-01 00:13:23" ...
    ##  $ end_time            : POSIXct, format: "2019-01-01 00:11:07" "2019-01-01 00:15:34" "2019-01-01 00:27:12" ...
    ##  $ bikeid              : int  2167 4386 1524 252 1170 2437 2708 2796 6205 3939 ...
    ##  $ tripduration        : num  390 441 829 1783 364 ...
    ##  $ from_station_id     : int  199 44 15 123 173 98 98 211 150 268 ...
    ##  $ from_station_name   : chr  "Wabash Ave & Grand Ave" "State St & Randolph St" "Racine Ave & 18th St" "California Ave & Milwaukee Ave" ...
    ##  $ to_station_id       : int  84 624 644 176 35 49 49 142 148 141 ...
    ##  $ to_station_name     : chr  "Milwaukee Ave & Grand Ave" "Dearborn St & Van Buren St (*)" "Western Ave & Fillmore St (*)" "Clark St & Elm St" ...
    ##  $ usertype            : chr  "Subscriber" "Subscriber" "Subscriber" "Subscriber" ...
    ##  $ gender              : chr  "Male" "Female" "Female" "Male" ...
    ##  $ birthyear           : int  1989 1990 1994 1993 1994 1983 1984 1990 1995 1996 ...
    ##  $ tripduration_minutes: num  6.5 7.35 13.82 29.72 6.07 ...

    # delete tripduration in sec
    clean_data <- all_trips %>%
      select(-tripduration)

    # round tripduration_minutes to 2 decimal places
    clean_data$tripduration_minutes <- round(clean_data$tripduration_minutes, 2)
    # check tripduration_minutes range
    range(clean_data$tripduration_minutes)

    ## [1]      1.02 177140.00

    # check unique labels for usertype
    unique(clean_data$usertype)

    ## [1] "Subscriber" "Customer"

    # check unique labels for gender
    unique(clean_data$gender)

    ## [1] "Male"   "Female" ""

    # check how many trips are over 24h; they may be outliers that can be neglected
    sum(clean_data$tripduration_minutes >= 1440)

    ## [1] 1849

    # trip over 24h percentage is 0.04%, can be treated as outliers and deleted from the dataset
    clean_data <- (clean_data %>%
                     filter(tripduration_minutes < 1440))

    # See the numeric summaries of the cleaned dataset
    summary(clean_data)

    ##     trip_id           start_time                        end_time                     
    ##  Min.   :21742443   Min.   :2019-01-01 00:04:37.00   Min.   :2019-01-01 00:11:07.00  
    ##  1st Qu.:22873766   1st Qu.:2019-05-29 15:47:49.00   1st Qu.:2019-05-29 16:07:30.00  
    ##  Median :23962244   Median :2019-07-25 17:49:19.00   Median :2019-07-25 18:08:30.00  
    ##  Mean   :23915593   Mean   :2019-07-19 21:44:54.03   Mean   :2019-07-19 22:03:56.25  
    ##  3rd Qu.:24963670   3rd Qu.:2019-09-15 04:29:04.00   3rd Qu.:2019-09-15 06:36:49.50  
    ##  Max.   :25962904   Max.   :2019-12-31 23:57:17.00   Max.   :2020-01-01 17:25:25.00  
    ##                                                                                      
    ##      bikeid     from_station_id from_station_name  to_station_id   to_station_name   
    ##  Min.   :   1   Min.   :  1.0   Length:3816155     Min.   :  1.0   Length:3816155    
    ##  1st Qu.:1727   1st Qu.: 77.0   Class :character   1st Qu.: 77.0   Class :character  
    ##  Median :3453   Median :174.0   Mode  :character   Median :174.0   Mode  :character  
    ##  Mean   :3380   Mean   :201.6                      Mean   :202.6                     
    ##  3rd Qu.:5046   3rd Qu.:289.0                      3rd Qu.:291.0                     
    ##  Max.   :6946   Max.   :673.0                      Max.   :673.0                     
    ##                                                                                      
    ##    usertype            gender            birthyear      tripduration_minutes
    ##  Length:3816155     Length:3816155     Min.   :1759     Min.   :   1.02     
    ##  Class :character   Class :character   1st Qu.:1979     1st Qu.:   6.85     
    ##  Mode  :character   Mode  :character   Median :1987     Median :  11.82     
    ##                                        Mean   :1984     Mean   :  19.03     
    ##                                        3rd Qu.:1992     3rd Qu.:  21.37     
    ##                                        Max.   :2014     Max.   :1439.75     
    ##                                        NA's   :537801

## **Analyze and Share**

    # Calculate total trip counts for both user types
    total_trips <- clean_data %>%
      group_by(usertype) %>%
      summarize(count = n())
    # Format the count with commas
    total_trips$count <- format(total_trips$count, big.mark = ",")
    print(total_trips)

    # Format count back to number
    total_trips$count_num <- as.numeric(gsub(",", "", total_trips$count))
    # Calculate percentages
    total_trips <- total_trips %>%
      mutate(percentage = count_num / sum(count_num) * 100)

    # Create pie chart with percentages
    ggplot(total_trips, aes(x = "", y = count, fill = usertype)) +
      geom_bar(stat = "identity", width = 1) +
      coord_polar(theta = "y") +
      labs(title = "Proportion of Total Trips by User Type") +
      geom_text(aes(label = paste0(usertype, "\n", count, "\n", round(percentage, 1), "%")),
                position = position_stack(vjust = 0.5), color = "white", size = 5) + 
      theme_void() +
      theme(plot.title = element_text(hjust = 0.5),
            legend.position = "none")

![](cyclistic_files/figure-markdown_strict/pie%20chart-1.png)

    # Calculate average trip duration for each user type
    avg_tripduration <- clean_data %>%
      group_by(usertype) %>%
      summarize(avg_tripduration_minutes = round(mean(tripduration_minutes, na.rm = TRUE), 1))

    # Print the result
    avg_tripduration

    # Bar chart for average trip duration by user type
    ggplot(data = avg_tripduration, aes(x = usertype, y = avg_tripduration_minutes, fill = usertype)) +
      geom_bar(stat = "identity") +
      geom_text(aes(label = paste(avg_tripduration_minutes, "min")), vjust = -0.5, size = 4, color = "black") +
      labs(title = "Average Trip Duration by User Type",
           x = "",
           y = "Average Trip Duration") +
      theme_minimal() +
      guides(fill = FALSE) +
      theme(axis.text.x = element_text(size = 12, face = "bold"),  # Adjust axis text properties
            axis.text.y = element_text(size = 12, face = "bold"),
            plot.title = element_text(hjust = 0.5))

![](cyclistic_files/figure-markdown_strict/bar%20chart%20avg%20trip%20duration-1.png)

    ggplot(clean_data, aes(x = tripduration_minutes, fill = usertype)) +
      geom_histogram(binwidth = 10, boundary = 0, position = "dodge", color = "black") +
      facet_wrap(~ usertype, scales = "free_x") +
      labs(title = "Trip Duration Distribution by User Type",
           x = "Trip Duration in Minutes",
           y = "Trip Count",
           fill = "User Type") +
      xlim(0, 90) +
      theme_minimal() +
      theme(
        plot.title = element_text(hjust = 0.5),
        legend.position = "NONE")

    ## Warning: Removed 76191 rows containing non-finite outside the scale range (`stat_bin()`).

![](cyclistic_files/figure-markdown_strict/trip%20duration%20distribution-1.png)

    # Trip durations for casual customers typically range between 10 and 30 minutes, whereas trip durations for subscribers mostly fall within the 0 to 20 minute range.

    # add a column that shows day of the week (1 for Sunday through 7 for Saturday)
    clean_data$day_of_week <- wday(clean_data$start_time, label = FALSE)

    # add a column that shows if it's weekend or weekday
    clean_data$weekday_or_weekend <- ifelse(clean_data$day_of_week %in% c(1, 7), "Weekend", "Weekday")

    # Calculate average trip counts and average trip duration for weekdays and weekends for subscribers and casual customers
    daily_summary <- clean_data %>%
      mutate(day = as.Date(start_time)) %>%  # Extract the day from start_time
      group_by(day, weekday_or_weekend, usertype) %>%
      summarize(
        daily_trip_count = n(),
        avg_daily_tripduration_minutes = mean(tripduration_minutes, na.rm = TRUE),
        .groups = 'drop'
      )

    # Calculate the average daily trip count and average daily trip duration for weekdays and weekends
    average_daily_summary <- daily_summary %>%
      group_by(weekday_or_weekend, usertype) %>%
      summarize(
        avg_daily_trip_count = round(mean(daily_trip_count), 0),
        avg_daily_tripduration_minutes = round(mean(avg_daily_tripduration_minutes, na.rm = TRUE), 1),
        .groups = 'drop'
      ) %>%
      arrange(usertype, weekday_or_weekend)

    # Print the average daily summary as one table
    print(average_daily_summary)

    # Insights from the table:
    # Customers tend to use the bike service more on weekends compared to weekdays, indicating potential leisure needs.
    # Subscribers show higher usage overall, particularly on weekdays, indicating potential commuting patterns.
    # Average trip durations are generally shorter for Subscribers compared to Customers across both weekdays and weekends.

    # Create bar chart with labels inside bars
    ggplot(average_daily_summary, aes(x = weekday_or_weekend, y = avg_daily_trip_count, fill = usertype)) +
      geom_bar(stat = "identity", position = position_dodge(width = 0.9)) +
      geom_text(aes(label = avg_daily_trip_count), 
                 position = position_dodge(width = 0.9),
                vjust = -0.5, size = 4, color = "black") +
      geom_text(aes(y = 0, label = usertype), 
                position = position_dodge(width = 0.9), 
                vjust = 1.2, size = 4, color = "black") +
      labs(title = "Average Daily Trips by User Type and Day Type",
           x = "",
           y = "Average Daily Trips") +
      theme_minimal() +
      theme(axis.text.x = element_text(size = 12, face = "bold"),  # Adjust axis text properties
            axis.text.y = element_text(size = 12, face = "bold"),
            legend.position = "none",
            plot.title = element_text(hjust = 0.5))

![](cyclistic_files/figure-markdown_strict/weekday/weekend%20bar%20charts-1.png)

    clean_data <- clean_data %>%
      mutate(hour = hour(start_time))

    # Calculate trip counts by hour, usertype, and weekday/weekend
    top_trip_counts <- clean_data %>%
      group_by(hour, usertype, weekday_or_weekend) %>%
      summarize(trip_count = n()) %>%
      arrange(desc(trip_count)) %>%
      group_by(weekday_or_weekend, usertype) %>%
      top_n(2, trip_count)  # Select top 2 trips for each usertype and weekday/weekend

    ## `summarise()` has grouped output by 'hour', 'usertype'. You can override using the
    ## `.groups` argument.

    # Create the histograms
    ggplot(clean_data, aes(x = hour, fill = usertype)) +
      geom_histogram(binwidth = 1, position = "dodge", color = "black") +
      facet_wrap(~ weekday_or_weekend + usertype, ncol = 2, scales = "free_x") +
      labs(title = "Trip Counts by Hour for Each User Type and Day Type",
           x = "Hour of Day",
           y = "Trip Count",
           fill = "User Type") +
      theme_minimal() +
      theme(strip.text = element_text(size = 12, face = "bold"),
            axis.text.x = element_text(size = 10),
            axis.text.y = element_text(size = 10),
            legend.position = "none",
            strip.placement = "outside",  # Place facet labels outside the plot area
            plot.title = element_text(size = 14, face = "bold"),
            panel.spacing = unit(1, "lines")) +
      theme(axis.title.x = element_text(margin = margin(t = 20))) +  # Adjust top margin for x-axis label
      geom_text(data = top_trip_counts, aes(label = hour, y = trip_count + 2), 
                position = position_dodge(width = 1), size = 3, vjust = -0.5, color = "black")

![](cyclistic_files/figure-markdown_strict/peak%20hours-1.png)

    # categorize the trips into four seasons
    clean_data <- clean_data %>%
      mutate(season = case_when(
        month(start_time) %in% c(12, 1, 2) ~ "Winter",
        month(start_time) %in% c(3, 4, 5) ~ "Spring",
        month(start_time) %in% c(6, 7, 8) ~ "Summer",
        month(start_time) %in% c(9, 10, 11) ~ "Fall"
      ))

    # # Group by season and usertype
    seasonal_summary <- clean_data %>%
      mutate(season = factor(season, levels = c("Spring", "Summer", "Fall", "Winter"))) %>%
      group_by(usertype, season) %>%
      summarize(
        total_trips = n(),
        avg_trip_duration_minutes = mean(tripduration_minutes, na.rm = TRUE)
      ) %>%
      arrange(usertype, season)

    ## `summarise()` has grouped output by 'usertype'. You can override using the `.groups`
    ## argument.

    print(seasonal_summary)

    # Plotting the grouped bar chart
    ggplot(seasonal_summary, aes(x = season, y = total_trips, fill = usertype)) +
      geom_bar(stat = "identity", position = "dodge") +
      labs(
        title = "Total Trips by Season and User Type",
        x = "Season",
        y = "Total Trips",
        fill = "User Type"
      ) +
      theme_minimal() +
      theme(
        axis.text.x = element_text(size = 12),
        axis.text.y = element_text(size = 12),
        legend.position = "bottom",
        plot.title = element_text(size = 14, face = "bold", hjust = 0.5)
      )

![](cyclistic_files/figure-markdown_strict/seasonal%20plot-1.png)

    popular_start_stations <- clean_data %>%
      group_by(from_station_name, usertype) %>%
      summarize(trip_count = n()) %>%
      arrange(usertype, desc(trip_count))

    ## `summarise()` has grouped output by 'from_station_name'. You can override using the
    ## `.groups` argument.

    top_start_stations <- popular_start_stations %>%
      group_by(usertype) %>%
      top_n(10, trip_count)

    # Now do some adjustments for smooth combination and plot
    # Rename station column
    top_start_stations <- top_start_stations %>%
      rename(station_name = from_station_name)

    # Add size column
    top_start_stations$size <- sqrt(top_start_stations$trip_count) * 2

    # Add a column for plots's label
    top_start_stations$usertype_start_end <- paste(top_start_stations$usertype, "_start", sep = "")

    print(top_start_stations)

    popular_end_stations <- clean_data %>%
      group_by(to_station_name, usertype) %>%
      summarize(trip_count = n()) %>%
      arrange(usertype, desc(trip_count))

    ## `summarise()` has grouped output by 'to_station_name'. You can override using the `.groups`
    ## argument.

    top_end_stations <- popular_end_stations %>%
      group_by(usertype) %>%
      top_n(10, trip_count)

    # Now do some adjustments for smooth combination and plot
    # Rename column
    top_end_stations <- top_end_stations %>%
      rename(station_name = to_station_name)

    # Add size column
    top_end_stations$size <- sqrt(top_end_stations$trip_count) * 2

    # Add a column for plot's label
    top_end_stations$usertype_start_end <- paste(top_end_stations$usertype, "_end", sep = "")

    print(top_end_stations)

    # combine top start and end stations for further visualization
    combined_stations <- union(top_start_stations, top_end_stations)

    ggplot(combined_stations, aes(x = station_name, y = usertype_start_end, size = size, fill = usertype)) +
      geom_point(shape = 21, color = "black", aes(fill = usertype), stroke = 1) +
      scale_size(range = c(3, 18)) +
      labs(
        title = "Top Stations by User Type",
        x = "Station",
        y = "User Type & Start/End",
        size = "Trip Count",
        fill = "User Type"
      ) +
      theme_minimal() +
      theme(
        axis.text.x = element_text(angle = 45, hjust = 1),
        axis.text.y = element_text(size = 12),
        legend.position = "NONE",
        plot.title = element_text(size = 14, face = "bold", hjust = 0.4)
      )

![](cyclistic_files/figure-markdown_strict/top%20station%20plot-1.png)

    # Insight: The top start and end stations are generally consistent within the same user type. However, there is little overlap in the top stations between different user types.

    # Clean the data for gender analysis.
    data_with_gender <- clean_data %>%
      filter(!is.na(gender) & gender != "")

    # Number and proportion of gender by User Type
    gender_counts <- data_with_gender %>%
      group_by(usertype, gender) %>%
      summarise(count = n()) %>%
      ungroup()

    ## `summarise()` has grouped output by 'usertype'. You can override using the `.groups`
    ## argument.

    gender_proportions <- gender_counts %>%
      group_by(usertype) %>%
      mutate(proportion = count / sum(count)) %>%
      ungroup()

    # Plot proportion of genders by user type
    ggplot(gender_proportions, aes(x = usertype, y = proportion, fill = gender)) +
      geom_bar(stat = "identity", position = "fill") +
      scale_fill_manual(values = c("Male" = "blue", "Female" = "red")) +
      scale_y_continuous(labels = scales::percent) +
      labs(title = "Proportion of Genders by User Type",
           x = "User Type",
           y = "Proportion",
           fill = "Gender") +
      geom_text(aes(label = paste0(gender, "\n", round(proportion, 2)*100, "%")),
                position = position_stack(vjust = 0.5), color = "white", size = 5) +
      theme(legend.position = "none",  # Remove legend
            plot.title = element_text(hjust = 0.5))  # Center plot title

![](cyclistic_files/figure-markdown_strict/gender-1.png)

    # clean the data for birthyear analysis
    data_with_birthyear <- clean_data %>%
      filter(!is.na(birthyear) & birthyear != "")

    # Plot age distribution for Subscriber and Customer
    ggplot(data_with_birthyear, aes(x = birthyear, fill = usertype)) +
      geom_histogram(binwidth = 10, boundary = 1940, position = "dodge", color = "black") +
      facet_wrap(~ usertype, ncol = 2, scales = "free_x") +
      labs(title = "Birth Year Distribution by User Type",
           x = "Birth Year",
           y = "Trip Count",
           fill = "User Type") +
      theme_minimal() +
      xlim(1940, 2010)

    ## Warning: Removed 1238 rows containing non-finite outside the scale range (`stat_bin()`).

![](cyclistic_files/figure-markdown_strict/birthyear-1.png)

    # Insight: The majority of users are born between 1980 and 2000. Casual customers are predominantly born between 1990 and 2000, while subscribers are mostly born between 1980 and 1990.

## **Key Findings**

### **User Demographics**

-   Among all trips, casual customers outnumber subscribers by three to
    one.
-   Male users outnumber female users in both groups, but casual
    customers have a higher proportion of female users compared to
    subscribers.
-   The majority of users are born between 1980 and 2000. Casual
    customers are predominantly born between 1990 and 2000, while
    subscribers are mostly born between 1980 and 1990.

### **Trip Duration and Frequency**

-   Casual customers typically have longer trip durations (10-30
    minutes) compared to subscribers (0-20 minutes).
-   Subscribers predominantly use Cyclistic on weekdays.
-   Casual users favor weekends.

### **Usage Patterns**

-   On weekdays, the peak hours for subscribers are 8 AM and 5 PM.
-   For casual customers, the peak hour on weekdays is 5 PM.
-   On weekends, the peak hours for both user types are after 12 PM.

### **Seasonal Trends**

-   Cyclistic is most popular during the summer and least popular in
    winter for both user types.

### **Popular Stations**

-   Popular stations differ significantly between user types.
    -   Casual customers frequent Streeter Dr & Grand Ave, Lake Shore Dr
        & Monroe St, and Millennium Park.
    -   Subscribers prefer Clinton St & Washington Blvd, Canal St &
        Adams St, and Clinton St & Madison St.

## **Act Phase**

### Recommendations

1.  **Target Younger Audiences:** Since casual users are predominantly
    born between 1990 and 2000, create marketing campaigns that resonate
    with younger demographics. Use social media platforms like
    Instagram, TikTok, and Snapchat where this age group is highly
    active.

2.  **Highlight Cost Savings:** Emphasize how subscribing can save money
    for frequent users, especially those who currently take longer trips
    as casual customers. Show comparative costs between paying per trip
    versus subscribing for unlimited shorter trips.

3.  **Offer Flexible Subscription Plans:** Introduce flexible
    subscription plans that cater to varying needs, such as weekend-only
    plans or limited ride packages, to appeal to casual users who may
    not ride daily.

4.  **Weekend Specials:** Since casual users favor weekends, create
    special weekend promotions for subscribers. Offer perks like
    extended ride times or discounts on weekend rides for subscribers.

5.  **Peak Hour Benefits:** Promote benefits for subscribers during peak
    hours, such as guaranteed bike availability or priority access
    during the busiest times.

6.  **Seasonal Promotions:** Launch seasonal campaigns that offer
    discounts or special benefits for new subscribers, especially during
    the summer when usage peaks. Highlight the advantages of being a
    subscriber during the high-demand season.

7.  **Station-Based Incentives:** Implement incentives at popular casual
    user stations (Streeter Dr & Grand Ave, Lake Shore Dr & Monroe St,
    and Millennium Park) to encourage sign-ups. For example, set up
    subscription kiosks with promotional offers or host events at these
    locations.
