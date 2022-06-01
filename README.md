# CS5010 Final Project - Group 12

Jenny Jang, 
Carol Moore, 
Elina Ribakova 

Github repository: https://github.com/Eribakova/CS5010

Relevant Juyter Notebooks:  
Datacleaning_Final.ipynb and Analysis_Final.ipynb

Link to Google Drive with all the Datasets mentioned in the report: https://drive.google.com/drive/folders/1af62oMJvt0VrC8fkSnE-8pq36EJRhDcc?usp=sharing

# Bikesharing: Insight into Public Precautions During the COVID-19 Pandemic

<img src="https://github.com/Eribakova/CS5010/blob/main/Bike.PNG">

## Objectives

This project explores how outdoor activity changed with COVID-19, with a focus on bikesharing.  In the Washington DC area, there were mixed policies about outdoor gatherings.  Many local parks were closed throughout the summer. However, recreational pathways remained accessible even where 6-foot social distancing was difficult if not impossible, and the major DC area trails remained crowded and even broke weekend records for usage in March 2020 as cases were escalating [1]. We examined the public's use of rental bikes to gauge how the public took precautions or increased their risks throughout 2020 to the present. 

Our central analytic question is whether bikesharing declined or increased in the DC metro area relative to the previous year. Many areas reported a spike in bicycle sales and bike sharing. For example, Chicago reported “unprecedented demand” in its bike-sharing system [2].  We wanted to see if DC experienced a similar spike or if public health restrictions and precautions caused a net reduction in usage.  If the public were consistently taking increased precautions in all aspects of their public behavior - including transportation and recreation - and we would expect bikesharing to decrease and for trips to be shorter, especially in more crowded areas.  The CDC's recommended guideline for social distancing of 6 feet was reiterated by outdoor recreation advocates (see, e.g., [3]).

Finally, we hope to explore how an analysis like this, if expanded, could be used in urban planning, transportation, and public health policies.  Briefly, we conclude that this kind of analysis could help tailor a pandemic response regarding bikesharing and other forms of outdoor recreation.

## Data set

We used Capital Bikeshare’s Trip History dataset which is licensed for public use by Motivate, the company that operates Capital Bike share on behalf of Washington, DC area municipalities.  Capital Bikeshare maintains over 4,300 bikes across DC, Maryland and Northern Virginia and is the dominant bike sharing company in the region.  The data are available at https://www.capitalbikeshare.com/system-data [4]. 

We downloaded 25 monthly data files from February 2019 to March 2021, the most recent file available.  Each record is a trip from a starting kiosk, or station, to an end station.  Data fields include starting and ending station address, a date-timestamp (year-month-day-hour-minute-second) and the type of renter, e.g., casual or membership.  In the most recent part of the dataset, longitude and latitude of the bike stations was also provided. 

Public health data provided helpful context for understanding bikesharing trends. We obtained epidemiological data from https://data.virginia.gov/Government/VDH-COVID-19-PublicUseDataset-Cases/bre9-aqqr [5]. The dataset provided information on hospitalizations and mortality rates by locale in Virginia. These data were merged in with the bikeshare data to complete our file.

## Data Pipeline

The Python code for preprocessing and cleaning is in the file Datacleaning_Final.ipynb.

### Preprocessing

In order to concatenate the 25 monthly bike share files into a single data set, we assessed if there were inconsistencies in the structure of the files over time.  We found that column headings and formats differed for files before and after May 2020. In most cases, however, the file format differences were not significant.  We ran each monthly dataframe through a for-loop to standardize headings, which allowed the files to be concatenated, resulting in a file of over 5.5M records. Once we standardized our data, we merged in daily COVID case counts by matching dates. Figures 1 and 2 show the standardization of column headings and the process for merging public health data.

![image](https://user-images.githubusercontent.com/70774260/117550168-ccfa8b80-b00c-11eb-9731-bbc6930c5a46.png)

![image](https://user-images.githubusercontent.com/70774260/117550185-ed2a4a80-b00c-11eb-85e2-c14fe658900e.png)

### Cleaning

The merged dataset included numerous irregularities and missing values that had to be remedied prior to analysis. We eliminated trips with outlying durations spanning multiple days, where likely someone did not return a bike.  The data set had street address information but lacked city and state, aggregates that we believed would be more closely tied to public health regulations and that would be more tractable to work with.  Examples of locations in raw data are:

- Dunn Loring Metro
- Fessenden St & Wisconsin Ave NW
- Shirlington Transit Center / Quincy St & Randolph St
- Shady Grove Metro West

We used the Python GEOPY library to determine the county, city and state from from which a bike share originated, based on latitude and longitude ("lat/long").  There were several issues which we solved in the following ways.

1.  Lat/long data were missing from all files dated prior to May 2020.  We created a dictionary to map lat/long to each station where data was available, then used the dictionary to fill in lat/long based on station id. Because the stations did not change much, if at all, during our timeframe, this was possible. 

2.  Lat/long data were not standard, differing greatly in the number of decimal places.  This resulted, for example, in over 48,000 unique lat/longs for fewer than 600 stations in the March 2021 file alone. As a result, and because we were trying to run GEOPY over every observation in the file, the reverse gecode runtime totaled over 3 hours for one month's data.  Our solution was to take the mean lat-long grouped by station id and run GEOPY over a small table that listed each station with the average lat/long associated with it.  Having cleaned and reduced the data, we easily extracted State and city information.  The breakdown of state is:

- District of Columbia:  4,892,84
- Virginia:  590,687
- Maryland: 135,035
- NaN: 119,252

Based on this information we decided to limit the project Virginia only, so that the final dataset consists of 590,687 bikesharing trips that started in the Northern Virginia locales of Arlington County, Alexandria City, Falls Church, and Fairfax County.  First-look analysis showed no patterns by county or city so we analyzed Virginia locales as a group.

## Experimental Design

Our approach was to compare bike sharing patterns 'pre-COVID' and 'post-COVID'.  Because bicycling is a seasonal activity we also decided to compare patterns month-by-month - for example, to compare June 2019 (pre-COVID) to June 2020 (post-COVID).  

There are several analytic limitations.  First, we don't know how the rider adjusted all aspects of their behavior to reduce risk - e.g., mask wear and diversion away from crowded trails to city streets. Anecdotally, mask wear was not common in DC during outdoor exercise particularly in the summer.  Secondly, Capital Bikeshare offered free memberships to essential workers during 2020, an additional source of demand with unknown implications for overall trip count and patterns of use. Finally, there were no bikeshares in April 2020 during the lockdown and as a result our dataset lacks any data for April.  

We planned four main queries:
1.  How did the number of monthly trips change in 2020 compared to 2019?
2.  Did the amount of time spent riding change?
3.  How did time of day and day of week change?  Do the shifts reflect changes in commuting, leisure, or both?
4.  What stations experienced change?  Is there a locational pattern?

The analysis and visualization code, along with additional visualizations, is in our Jupyter notebook.  An important coding tool was the Pandas 'groupby' method.  We used groupby to aggregate the data and develop monthly, weekly, daily and hourly counts.  The Seaborn library was also important to our analysis.  Seaborn easily makes bar charts that are useful in 'pre-/post-analysis' because the 'hue' keyword enables dimensions beyond the x and y axis.  This feature allowed us to highlight patterns by time period.

## Key results

### Bike sharing usage was down during COVID-19.
Bike sharing was comparatively low in 2020 due to COVID-19 (Figure 3). As infections picked up and lockdowns were announced, bike sharing dropped off and failed to pick up to the pre-COVID levels even by the end of 2020. It is notable that data suggests that people began taking precautions during the uncertain weeks before before Virginia's lockdown, since trips were lower in February than January 2020. 

![image](https://user-images.githubusercontent.com/70774260/117549581-37113180-b009-11eb-8da3-1cbdb78c3b52.png)

### Bike sharing shifted from commuting hours to other times of day.
Trips shifted from commuting hours to afternoons and evenings. However, the change did not occur immediately. Trips were down throughout the day in March 2020 compared to March 2019.  However, there were still a lot of trips during commuting hours. By June, commuting hour trips were much lower than in 2019 and late afternoon trips were up. The finding suggests that after the lockdowns in April, more people might have settled into working from home, but continued to use bike share, this time for leisure.  Our display (Figure 4) shows March and June 2020; visualizations in our Jupyter Notebook confirm the trend continued after June.

![image](https://user-images.githubusercontent.com/70774260/117549626-7b043680-b009-11eb-9f79-b39dab864536.png)

### Weekend trips increased.
Results by day of the week also suggest an increase in recreational use and a decrease in commuting use. In March 2020 trips were way down every day of the week. By June, there were ~500 more weekend trips than in 2019. Finally, by early 2021, Saturday trips appear to have fully caught up with pre-COVID-19 levels. Our display (Figure 5) shows March and June 2020; other visualizations confirm the trend continued after June.

![image](https://user-images.githubusercontent.com/70774260/117549786-5197da80-b00a-11eb-9d30-f5e559121ef0.png)

### Duration of trips increased.  
People take longer bike rides suggesting that they are riding bikes more for leisure or extended personal use rather than commuting to or from work. The distribution of duration by time of day is showin in Figure 6.

![image](https://user-images.githubusercontent.com/70774260/117549978-89535200-b00b-11eb-9782-f3e3658bb504.png)

### Trips from recreational sites were relatively more frequent.  
Bikeshare stations are concentrated near commuting hubs and recreational areas. The top 20 most used stations changed markedly since the start of the pandemic (Figure 7). The popular rental sites Gravelly Point and Roosevelt Island are on the Mount Vernon Trail, a 6-foot wide path on which maintaining the CDC's recommended 6 foot social distancing is not possible.  Popular side routes and adjacent trails are also narrow. The number of trips starting from these stations during 2020 was the same as during 2019. 

![image](https://user-images.githubusercontent.com/70774260/117550102-5cec0580-b00c-11eb-8e40-51a5b32499e6.png)

## Testing of our program 

We use method-based unit testing for our data cleaning as well as the data analysis. Below are a few examples of the tests we performed. 
* We checked that our lat/long information is correctly matched to stations where it was missing (earlier part of the data sample). 
* We recalculated the duration of the trip and made sure it is correctly reflected in our dataset. 
* We used assert statements to make sure the groupby split-apply-combine result was as intended (note that this approach yields no output if the statement is True) - examples in Figure 9.

![image](https://user-images.githubusercontent.com/70774260/117549451-755a2100-b008-11eb-8d02-80fb248964ce.png)

## Conclusions

We used high frequency data on bike sharing before and after the onset of COVID-19 in DC metropolitan area, focusing on Northern Virginia specifically. We found that the overall usage of bike sharing dropped off significantly as the rate of infections picked up. However, after the Virginia's lockdown ended in early May, bike sharing continued its seasonal increase even as deaths and hospitalizations escalated, and surpassed the previous year during non-commuting hours and on weekends.  Together, our data suggests a need for increased public health educaton and precautions governing outdoor recreational sites during popular leisure hours, especially given bikesharing was even higher on weekends during the pandemic than during the prior year. 

Public health officials could the data to tailor interventions beyond what the vendor is already doing (sanitizing high-contact parts of the equipment after use). These could include posting safety information posters at the bikesharing stations that are most frequently used, requiring a review of social distancing guidelines as part of the terms of the rental, emphasizing the value of mask wear for outdoor recreation, or limiting rentals where social distancing is not possible. 

Urban planners could use this information to assess where and when roads might be closed promote fitness and recreation, to enable more social distancing than is possible on trails.  By monitoring frequency of trips during certain days or hours of the day, they can adjust road closures, particularly on the weekends, to relieve bike congestion on crowded paths where social distancing is difficult if not impossible. 

There are a few technical limitations we would overcome with more time.  For example, rather than simply omitting April 2020, we would treat April 2020 as a month with zero bikeshares, and include public health case data for that month.  Doing so would have enabled a greater variety of analyses, such as line charts showing correlations between public health events and bike shares.  

To increase the utility of this analysis, we would:
- Develop a user query interface with mapping functionality to show the most frequently used stations by time of day and day of the week. This would make it easier for government officials to assess when and where to dedicate more resources to public education campaigns. 
- Develop an algorithm to identify more stations that are sited on recreational trails those near commuting hubs.  This enable us to better distinguish between leisure and work-related use. 
- Seek additional data on essential workers who took advantage of Capital Bikeshare's free membership program, to assess the potential impact of the program and also to separate those trips from what appear to be leisure trips.  

![image](https://user-images.githubusercontent.com/70774260/117548484-442b2200-b003-11eb-8f06-c2fb24eadbf0.png)


## References

#### Cited

[1] Dunbar, Harry, "How to Avoid Crowded Trails During COVID-19", BikeArlington blog post, April 24, 2020. https://www.bikearlington.com/how-to-avoid-crowded-trails-during-covid-19/

[2] Kapp, Amy, "Here’s the Latest Expert Guidance on Outdoor Activity and COVID-19," Rails to Trails Conservancy blog post, April 18, 2020, https://www.railstotrails.org/trailblog/2020/april/18/here-s-the-latest-expert-guidance-on-outdoor-activity-and-covid-19/

[3] Chicago Metropolitan Agency for Planning (CMAP), "Pandemic presents opportunity for communities to embrace biking and walking," website post, undated).https://www.cmap.illinois.gov/updates/all/-/asset_publisher/UIMfSLnFfMB6/content/pandemic-presents-opportunity

[4] Data license agreement.  https://www.capitalbikeshare.com/data-license-agreement

[5] Commonwealth of Virginia, "Virginia Open Data Portal:  VDH-COVID-19-PublicUseDataset-Cases", website, https://data.virginia.gov/Government/VDH-COVID-19-PublicUseDataset-Cases/bre9-aqqr

#### General

https://dtkaplan.github.io/DataComputingEbook/index.html#table-of-contents

https://towardsdatascience.com/visualizing-bike-mobility-in-london-using-interactive-maps-for-absolute-beginners-3b9f55ccb59

https://towardsdatascience.com/exploring-bike-share-data-3e3b2f28760c

https://towardsdatascience.com/applied-exploratory-data-analysis-the-power-of-visualization-bike-sharing-python-c5b2645c3595

https://seaborn.pydata.org/generated/seaborn.violinplot.html
