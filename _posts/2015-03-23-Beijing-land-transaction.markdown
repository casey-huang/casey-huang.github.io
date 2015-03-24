---
layout: post
title:  "My first blogpost - Beijing Land Transaction"
date:   2015-03-23 22:34:25
categories: jekyll update
tags: featured
image: /assets/article_images/2015-03-23-Beijing-land-transaction/beijing_watercolor.jpeg
---

##Beijing Land Transaction 2003-2013
===================

###Source:

I was curious about visualization and geocoding work people have done in China, and I stumbled upon this [Beijing City Lab](http://www.beijingcitylab.com/). The Lab aims to quantify urban dynamics and generates new insights for urban planning and governance, with a focus on the capital Beijing.

While the projects are pretty impressive, I found the cover map of [Data 2](http://www.beijingcitylab.com/data-released-1/data1-20/) could be more informative. 

It looks like this:
![](http://casey-huang.github.io/assets/article_images/2015-03-23-Beijing-land-transaction/original_image.jpg)

###What's wrong:

Dot plot on a map is not a good choice here:
> - The details of transactions besides geo-location are all left out, merely any information
> - When the dots are overlapped with each other, even density is hard to tell
> - Impossible to observe trends

###First look at the dataset:

After downloading the [shapefiles](https://www.dropbox.com/s/gc93o1iz7mcu6j9/DT2.rar), I want to use github's support with .geojson files to get a close look of data. Here's how I convert it:

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install gdal
ogr2ogr -f GeoJSON Downloads/beijing1.geojson Downloads/DT2geo/BJ_Land_Transactions.shp
```

<script src="https://embed.github.com/view/geojson/casey-huang/casey-huang.github.io/master/beijing1.geojson"></script>

=======================

#####Questions the audience may be interested in and how I'd present the graph:

####1. For the most recent transactions in year 2013, how do land unit prices of different land usage compare to each other?

First, some necessary data munging:

```
# read the shapefile and convert to data frame
library(rgdal)
bj <- readOGR(dsn="Downloads/DT2geo",layer="BJ_Land_Transactions")
bj <- as(bj, "data.frame")

# eliminate irrelevant columns
bj = subset(bj, select = -c(城市,宗地四至,签约时间,合同约定开,合同约定竣,浏览次数,PageUrl,coords.x1,coords.x2))

#change the names of the rest to English
colnames(bj) <- c('obj_id', 'land_id','transferee','location','district','plot_area','archi_area',
                  'use','price','floor_area_ratio','announce_date','y','x')

# remove records where 0 m^2 land was transfered
bj = subset(bj, plot_area!=0)

# adding column "unit price" (RMB/m^2)
bj$unit_price = 100000 * bj$price / bj$plot_area
```

The description in "use" is pretty messy now, I want to classify them into top categories and other.

```
library(sqldf)
industrial=sqldf("SELECT * FROM bj WHERE use LIKE '工业%'")
residential=sqldf("SELECT * FROM bj WHERE use LIKE '住宅%' OR use LIKE '城镇住宅%' OR use LIKE '居住%' OR use LIKE '公寓%'")
commercial=sqldf("SELECT * FROM bj WHERE use LIKE '商业%' OR use LIKE '商务%' OR use LIKE '商服%'")
office=sqldf("SELECT * FROM bj WHERE use LIKE '办公%'")
multiple=sqldf("SELECT * FROM bj WHERE use LIKE '综合%'")

others =sqldf("SELECT * FROM bj R WHERE R.obj_id not in (Select obj_id from 'industrial')")
others =sqldf("SELECT * FROM others R WHERE R.obj_id not in (Select obj_id from 'residential')")
others =sqldf("SELECT * FROM others R WHERE R.obj_id not in (Select obj_id from 'commercial')")
others =sqldf("SELECT * FROM others R WHERE R.obj_id not in (Select obj_id from 'office')")
others =sqldf("SELECT * FROM others R WHERE R.obj_id not in (Select obj_id from 'multiple')")

# add clean tags to each class and re-combine the dataset
land_use=c('residential','industrial','commercial','office','multiple','others')
residential$class=land_use[1]
industrial$class=land_use[2]
commercial$class=land_use[3]
office$class=land_use[4]
multiple$class=land_use[5]
others$class=land_use[6]
# join all the classes to get a new data frame
bj_all = rbind(residential, industrial, commercial, office, multiple, others)
```

Make descending bar chart of median(unit_price) ~ class within 2013:

```
all_2013 = subset(bj_all, as.Date(announce_date)>'2013-01-01')
library(ggplot2)
plotdf=aggregate(all_2013$unit_price, by=list(all_2013$class), FUN='median')
colnames(plotdf)=c('land_use_2013','median_unit_price')
plotdf$land_use <- with(plotdf, reorder(land_use_2013, -median_unit_price))
p1 = ggplot(plotdf, aes(y=median_unit_price, x=land_use_2013)) + geom_bar(stat = "identity")
p1
```

![](http://casey-huang.github.io/assets/article_images/2015-03-23-Beijing-land-transaction/unit_class.png)

Observe that the class "others" has the largest median price in 2013, after checking the original usage and unit prices, I think it is because the variety of function of land here, including airports, hotels&restaurants, public utility, education, tech R&D, etc.

####2. The movement of unit price and #transactions of different lands throughout 10 years?

For unit price line, I didn't take each data point, because the extremity in price (a lot of 0 cost land and several super high cost) doesn't reflect the general trend, so I take the median of each class each year.

```
# number of land transactions each year by land
library(plyr)
count=count(bj_all, c("year", "class"))
colnames(count)=c('year','class','number_of_transactions')
p2 = ggplot(count, aes(x=year, y=number_of_transactions, group = class, colour = class)) + geom_line()

# unit price of different land each year
library(reshape2)
bj_all$year = format(as.Date(bj_all$announce_date), "%Y")
plotdf2=aggregate(bj_all$unit_price, by=list(bj_all$class, bj_all$year), FUN='median')
colnames(plotdf2)=c('class','year','median_unit_price')
p3 = ggplot(plotdf2, aes(x=year, y=median_unit_price, group = class, colour = class)) + geom_line()

library(gridExtra)
grid.arrange(arrangeGrob(p2, p3))
```

![](http://casey-huang.github.io/assets/article_images/2015-03-23-Beijing-land-transaction/by_year.png)

Observe that there's sharp drop in number of transactions of residential land in between 2004 and 2005. This should be due to the large amendment of land management law that took place in 2004.

####3. Popular location to build a new store/shopping mall?

Plot a static map using ggmap:

```
commercial_2011 = subset(commercial, as.Date(announce_date)>'2011-01-01', select=c('Y','X'))
library(ggmap)
bjbase = qmap(location= "Beijing", zoom = 11, maptype = "road", source = "google")
p4 = bjbase + 
  stat_density2d(data = commercial_2011,
                 aes(x = X, y = Y, fill = ..level.., alpha = ..level..),
                 size = 15, bins = 8, geom = "polygon") + 
  scale_alpha(range = c(0.3, 0.7), guide = FALSE) +
  scale_fill_gradient(low = "beige", high = "red")
p5 = bjbase + 
  stat_bin2d(data = commercial_2011,
             aes(x=X, y=Y, fill=cut(..count.., c(0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20)), bins = 20),
             alpha = 0.4)+
  scale_fill_hue("count")
p4
p5
```

![](http://casey-huang.github.io/assets/article_images/2015-03-23-Beijing-land-transaction/p4.png)
![](http://casey-huang.github.io/assets/article_images/2015-03-23-Beijing-land-transaction/p5.png)
The two maps tells the same story, and the populat spots for industrial or residential development can be drawn similarly too.

####4 Interactive map?

I intended to make a choropleth that change with time, and struggled using rMaps, since the feature of sliding time bar to view the district choropleth over time is desired. Since there's some issue I can't fix, I use CartoDB as an alternative:

<iframe width='100%' height='520' frameborder='0' src='http://caseyhuang.cartodb.com/viz/35ace442-d234-11e4-a7aa-0e853d047bba/embed_map' allowfullscreen webkitallowfullscreen mozallowfullscreen oallowfullscreen msallowfullscreen></iframe>

=================

###Difficulties and more to-be-worked-on:

> - Missing data and messy preprocessing. Missing data is annoying , but the inconsistent preprocessing make things worse. By looking at the dataset, I would tell that not all missing data are marked as "NA", some of them are recorded as 0 (as in floor_area_ratio, and plot_area etc.).
> - In the bin map, since the "fill" is decided by bins, I didn't find a way to change coloring to the color look more intense for higher count.
> - rMaps or other tools to make choropleth with sliding time.

###Some other thoughts:

Compare to the original graph which is over-simplified, some complicated interactive map seems not necessary to me. For example, the ones in [Gallup Analytics](http://www.gallup.com/poll/125066/State-States.aspx) only have two columns of data. 

It also has a major problem of coloring every map in green. For example, in "Feel active and productive", the greener the state is, the well-being is higher. This agrees with my instinct when seeing greener color. But the site also uses more green to show more people are worried about money too, which is somehow misleading.
