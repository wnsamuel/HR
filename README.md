---
title: "HR Movement dashboard"
output: 
  flexdashboard::flex_dashboard:
    orientation: rows
    vertical_layout: fill
    runtime: shiny
    source_code: embed
    
---

```{r setup, include=FALSE}
library(flexdashboard)
library(ggplot2)
library(readxl)
library(tidyverse)
library(plotly)
library(ggmap)

devtools::install_github('rstudio/rmarkdown@3953abd473e9230faca4dcad996d5d1b35c3b3a7')

#Use knitr root.dir instead of setwd to set directory
#It is not working
require(knitr)
#opts_knit$set(root.dir = "C:\\CTS Learning\\02_CDO - Certified Data Analyst\\HR dashboard project\\Regional files")

#Reading regional files
americas=read_excel("C:\\Users\\SB\\Documents\\R\\Regional files\\HR Dashboard_v1 - Americas.xlsx",sheet = "data")

apac=read_excel("C:\\Users\\SB\\Documents\\R\\Regional files\\HR Dashboard_v1 - APAC.xlsx",sheet = "data")

europe=read_excel("C:\\Users\\SB\\Documents\\R\\Regional files\\HR Dashboard_v1 - Europe.xlsx",sheet = "data")


#Consolidating global data
global=rbind(americas,apac,europe)

#Read file containing country coordinates
country_coor=read_excel("C:\\Users\\SB\\Documents\\R\\country coord.xlsx")

```


Dash {data-icon="fa-globe"}
=============================
Row {data-width=150}
--------------------------------------
### Total new hires
```{r}
newhires=global %>% filter(`Type of movement`=="Entry") %>% nrow()
valueBox(value = newhires,icon = "fa-user-plus",caption = "New Hires",color = "green")
```

### Total exits
```{r}
exits=global %>% filter(`Type of movement`=="Exit") %>% nrow()
valueBox(value = exits,icon = "fa-user-times",caption = "Exits", color = "orange")
```

### Net change
```{r}
newhires=global %>% filter(`Type of movement`=="Entry") %>% nrow()
exits=global %>% filter(`Type of movement`=="Exit") %>% nrow()

netchange=(newhires-exits)

#If loop to have either up-arrow or down-arrow icon on valuebox based on the value of netchange
if(netchange>0){
  valueBox(value = netchange,icon = "fa-arrow-up",caption = "Net Change", color = "coral")
} else{
valueBox(value = netchange,icon = "fa-arrow-down",caption = "Net Change", color = "coral")}

```


Row
----------------------------------

### Movement by month
```{r}
h1=global %>% group_by(Month,`Type of movement`) %>% 
  summarise(count=n())

p1=plot_ly(data = h1,
        x=h1$Month,
        y=h1$count) %>% 
  add_lines(linetype = h1$`Type of movement`,
            data = h1$count,
            hoverinfo="text",
            text=paste(h1$count)) %>% 
  layout(xaxis=list(title="Month"),
         yaxis=list(title="Count")) 
p1

```

### Movement by Region
```{r}
#Count by month, entry/exit and country
h2=global %>% 
  group_by(Month,`Type of movement`,Country) %>% 
  summarise(count=n())

#Add long/lat info based on country name from countr_coor
h2=left_join(h2,country_coor[,2:4],by=c("Country"="name"))
#h2 is the table where I want the info
#country_coor[,2:4] selects only long, lat and name columns
# so that I can only add the long and lat cols to h2 after the join
#by=c() identifies the keys in both dataframes

p2=plot_geo(h2,locationmode="world") %>% 
  add_markers(x=h2$longitude,
              y=h2$latitude,
              size=h2$count,
              color=h2$`Type of movement`,
              hoverinfo="text",
              hovertext=paste(h2$`Type of movement`,": ",h2$count)) %>% 
  layout()


p2

```

New hires {data-icon="fa-user-plus"}
==================
Row{data-height=250}
--------

### Avg New Hires YTD
```{r}
newhire_bymonth=global %>% 
  filter(`Type of movement`=="Entry") %>% 
  group_by(Month) %>% 
  summarise(count=n())

avgnewhire=round(mean(newhire_bymonth$count),2)
  
valueBox(avgnewhire,icon = "fa-user-plus",caption = "Average monthly new hires",color = "green")
```

### New hire split by Employment
```{r}
h5=global %>% 
  filter(`Type of movement`=="Entry") %>% 
  group_by(`Employment type`) %>% 
  summarise(count=n())

p5=plot_ly(h5) %>% 
  add_pie(labels=h5$`Employment type`,values=h5$count,hole=0.6)

p5
```


### New hire split by Work Authorisation
```{r}
h6=global %>% 
  filter(`Type of movement`=="Entry") %>% 
  group_by(`Work Authorisation`) %>% 
  summarise(count=n())

p6=plot_ly(h6) %>% 
  add_pie(labels=h6$`Work Authorisation`,values=h6$count,hole=0.6)

p6

```


Row
----------

### New hires by Country
```{r}

#Summarise and group by Country
h3=global %>% 
  filter(`Type of movement`=="Entry") %>% 
  group_by(Month,Country) %>% 
  summarise(count=n())

#Use spread to make the table ready for plots
h3=spread(h3,key = Country,value = count)


#Bar chart by country
p3=plot_ly(h3,
           x=h3$Month,
           hoverinfo="text") %>% 
  add_bars(y=h3$Argentina,
           name="Argentina",
           hovertext=paste(h3$Argentina)) %>% 
  add_bars(y=h3$Australia,
           name="Australia",
           hovertext=paste("Australia: ",h3$Australia)) %>% 
  add_bars(y=h3$Brazil,
           name="Brazil",
           hovertext=paste("Brazil: ",h3$Brazil)) %>%
  add_bars(y=h3$Canada,
           name="Canada",
           hovertext=paste("Canada: ",h3$Canada)) %>%
  add_bars(y=h3$India,
           name="India",
           hovertext=paste("India: ",h3$India)) %>%
  add_bars(y=h3$Romania,
           name="Romania",
           hovertext=paste("Romania: ",h3$Romania)) %>%
  add_bars(y=h3$USA,
           name="USA",
           hovertext=paste("USA",h3$USA))
  
  
p3
```


### New hires by Org
```{r}
#Create a summary grouped by Org
h4=global %>% 
  filter(`Type of movement`=="Entry") %>% 
  group_by(Month,Org) %>% 
  summarise(count=n())

#Make the table plot-ready
h4=spread(h4,key = Org,value = count)

p4=plot_ly(data = h4,
           x=h4$Month,
           hoverinfo="text") %>% 
  add_bars(y=h4$Corporate,
           name="Corporate",
           hovertext=paste("Corporate: ",h4$Corporate)) %>% 
  add_bars(y=h4$Delivery,
           name="Delivery",
           hovertext=paste("Delivery: ",h4$Delivery)) %>% 
  add_bars(y=h4$DeliveryOps,
           name="DeliveryOps",
           hovertext=paste("DeliveryOps: ",h4$DeliveryOps)) %>% 
  add_bars(y=h4$Support,
           name="Support",
           hovertext=paste("Support: ",h4$Support))
p4
```






Attrition {data-icon="fa-user-times"}
====================

Row{data-height=250}
--------

### Avg Attrition YTD
```{r}
attn_bymonth=global %>% 
  filter(`Type of movement`=="Exit") %>% 
  group_by(Month) %>% 
  summarise(count=n())

avgattn=round(mean(attn_bymonth$count),2)
  
valueBox(avgattn,icon = "fa-user-times",caption = "Average monthly attrition",color = "orange")
```

### Attrition split by Employment
```{r}
h7=global %>% 
  filter(`Type of movement`=="Exit") %>% 
  group_by(`Employment type`) %>% 
  summarise(count=n())

p7=plot_ly(h7) %>% 
  add_pie(labels=h7$`Employment type`,values=h7$count,hole=0.6)

p7
```


### Attrition split by Work Authorisation
```{r}
h8=global %>% 
  filter(`Type of movement`=="Exit") %>% 
  group_by(`Work Authorisation`) %>% 
  summarise(count=n())

p8=plot_ly(h8) %>% 
  add_pie(labels=h8$`Work Authorisation`,values=h8$count,hole=0.6)

p8

```


Row
----------

### Attrition by Country
```{r}

#Summarise and group by Country
h9=global %>% 
  filter(`Type of movement`=="Exit") %>% 
  group_by(Month,Country) %>% 
  summarise(count=n())

#Use spread to make the table ready for plots
h9=spread(h9,key = Country,value = count)

#I'm trying to understand how to solve the issue of having dynamic add_bars()for plotly
#In the below snippet, I have commented bars for Argentina, BRazil and Canada because there are no exits for these locations
# and I am getting the error: "Must supply x/y attributes" 


#Bar chart by country
p9=plot_ly(h9,
           x=h9$Month,
           hoverinfo="text") %>% 
  # add_bars(y=h9$Argentina,
  #          name="Argentina",
  #          hovertext=paste(h9$Argentina)) %>% 
  add_bars(y=h9$Australia,
           name="Australia",
           hovertext=paste("Australia: ",h9$Australia)) %>% 
  # add_bars(y=h9$Brazil,
  #          name="Brazil",
  #          hovertext=paste("Brazil: ",h9$Brazil)) %>%
  # add_bars(y=h9$Canada,
  #          name="Canada",
  #          hovertext=paste("Canada: ",h9$Canada)) %>%
  add_bars(y=h9$India,
           name="India",
           hovertext=paste("India: ",h9$India)) %>%
  add_bars(y=h9$Romania,
           name="Romania",
           hovertext=paste("Romania: ",h9$Romania)) %>%
  add_bars(y=h9$USA,
           name="USA",
           hovertext=paste("USA",h9$USA))
  
  
p9
```


### Attrition by Org
```{r}
#Create a summary grouped by Org
h10=global %>% 
  filter(`Type of movement`=="Exit") %>% 
  group_by(Month,Org) %>% 
  summarise(count=n())

#Make the table plot-ready
h10=spread(h10,key = Org,value = count)

p10=plot_ly(data = h10,
           x=h10$Month,
           hoverinfo="text") %>% 
  add_bars(y=h10$Corporate,
           name="Corporate",
           hovertext=paste("Corporate: ",h10$Corporate)) %>% 
  add_bars(y=h10$Delivery,
           name="Delivery",
           hovertext=paste("Delivery: ",h10$Delivery)) %>% 
  add_bars(y=h10$DeliveryOps,
           name="DeliveryOps",
           hovertext=paste("DeliveryOps: ",h10$DeliveryOps)) %>% 
  add_bars(y=h10$Support,
           name="Support",
           hovertext=paste("Support: ",h10$Support))
p10
```

