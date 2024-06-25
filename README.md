# Case Study - Private Water Trucking in Syria

## Objective
By using the dataset of water trucking in Syria from REACH (https://repository.impact-initiatives.org/document/impact/f68bd982/REACH_SYR_Dataset_Water-Trucking_Mar24.xlsx), this case study is to examine the cost and effectiveness of private water trucking in Northwest Syria (NWS).

Data cleaning, data wrangling, data analysis and data visualization will be performed, using R, in order to know the cost per quantity of water in private water trucking activities. This case study will be focusing on the fuel cost for delivery of water.

## Data cleaning
1. Remove duplicate data

Using distinct() in R.

2. Remove extra spaces in each cell

Using str_rm_whitespace_df() in R.

3. Remove data cells with NA in currency and fuel_unit columns

Without currency and fuel unit, the cost is not measurable and comparable with each other so the data with currency and fuel unit of NA will be removed.

4. Check if the units for currency, fuel_unit and delivery distance are the same for all uuid.

If they have different units within the same variable, a conversion of unit should be applied to unify the units. The results show that the units are unified.

![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/currency.PNG)
![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/fuel_unit.PNG)
![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/truck_volume_unit.PNG)

5. Check if the calculation of fuel_cost_litre is accurate

Add a column 'cal_fuel_cost_litre' for calculation of fuel_cost_litre. Add another column 'cal_verify' to return TRUE if the result of cal_fuel_cost_litre and fuel_cost_litre match.

A summary of 'check_cal_fuel_cost_litre' shows if there are unmatched values of cal_fuel_cost_litre and fuel_cost_litre. The calcuation of fuel_cost_litre is verified as accurate.

![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/check_cal_fuel_cost_litre.png)

```r
# install.packages
install.packages("dplyr")
library(dplyr)
install.packages("tidyverse")
library(tidyverse)
library(readxl)
install.packages("hablar")
library(hablar)
install.packages('base')
library('base')
install.packages("forstringr")
library(forstringr)


#load data file
setwd("C:/Users/danny/Desktop/Tina/Syria")
main_data <- read_excel("REACH_SYR_Dataset_Water-Trucking_Mar24.xlsx", "Main Data")

#Data cleaning
#remove duplicates and extra spaces
main_data <- distinct(main_data)
main_data <- str_rm_whitespace_df(main_data)

#remove currency and fuel unit with a value of NA
main_data <- main_data[!is.na(main_data$currency),]
main_data <- main_data[!is.na(main_data$fuel_unit),]

#check if currency and fuel unit are the same for all data. result showed all currency and fuel unit were the same
currency <- main_data %>% group_by(currency) %>% count(currency)
view(currency)

fuel_unit <- main_data %>% group_by(fuel_unit) %>% count(fuel_unit)
view(fuel_unit)

truck_volume_unit <- main_data %>% group_by(truck_volume_unit) %>% count(truck_volume_unit)
view(truck_volume_unit)

#Check if the calculation of fuel_cost_litre is accurate
check_cal_fuel_cost_litre <- private_data %>% mutate(cal_fuel_cost_litre = round(cost_fuel_delivery1
/fuel_delivery)) %>%
  mutate(cal_verify = print (cal_fuel_cost_litre == cal_fuel_cost_litre)) %>% 
  group_by(cal_verify) %>%
  select(uuid, cal_verify)

view(check_cal_fuel_cost_litre)
summarise(check_cal_fuel_cost_litre)
```

## Data wrangling
We will create 2 tables for examining the fuel cost of delivering water in private water trucking activies.

The 1st table 'fuel_water_cost' will contain ki_type of 'private_trucker' or 'private_owner'. 4 variables needed for calculation of fuel cost (TRY) per a litre of water are extracted from the cleaned data.
A new column 'delivery_volume_litre' is added to convert barrel unit to litre unit.
An other new collumn 'fuel_delivery_costTRY_per_waterLitre' presents the fuel cost per liter of water (TRY/Litre).
Table 'fuel_water_cost' was cleaned agiin with na.omit() to remove NA value.

```r
#Data wrangling

#use filter to obtain data related to private water trucking
#calculate fuel cost per Lite of water
fuel_water_cost <- filter(main_data, ki_type == 'private_trucker' | ki_type == 'private_owner') %>%
  select(cost_fuel_delivery1, delivery_volume, delivery_distance) %>%
  mutate(delivery_volume_litre = delivery_volume*158.987294928) %>%
  mutate(fuel_delivery_costTRY_per_waterLitre = cost_fuel_delivery1/delivery_volume_litre) %>%
  na.omit(fuel_delivery_costTRY_per_waterLitre)
```

The 2nd table 'fuel_water_cost_summary' is a summary of table 'fuel_water_cost', showing the mean, max and min of the fuel cost per litre of water.

![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/fuel_water_cost_summary.PNG)
  
```r
#summary of the mean, minimum and maximum of the fuel cost for delivery per litre water 
fuel_water_cost_summary <- data.frame(Mean.of.fuel.cost.per.litre.water = mean(fuel_water_cost$fuel_delivery_costTRY_per_waterLitre), Min.of.fuel.cost.per.litre.water = min(fuel_water_cost$fuel_delivery_costTRY_per_waterLitre), Max.of.fuel.cost.per.litre.water = max(fuel_water_cost$fuel_delivery_costTRY_per_waterLitre)) 
```

## Data visualization

A scatter plot 'Fuel cost for delivery of water according to delivery distance' is created to see the correction of the fuel cost of water delivery and delivery distance. 

![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/Fuel%20cost%20for%20delivery%20of%20water%20according%20to%20delivery%20distance.png)

```r
#Data visualization
ggplot(fuel_water_cost, aes(x=delivery_distance, y=fuel_delivery_costTRY_per_waterLitre)) + 
  geom_point() +
  geom_smooth(method=lm) +
  labs(title="Fuel cost for delivery of water according to delivery distance",
       x="Delivery distance (km)", y = "Fuel cost per water (TRY/Litre)")

```

A Pearson correlation test is performed to obtain the p-value. p-value = 0.6764. Correlation coefficient = -0.04707941

![paste to excel](https://github.com/tinatmyiu/casestudy/blob/main/pearson%20correation.PNG)

```r
#Pearson correlation
fuel_water_cost %>%
  cor.test( ~delivery_distance + fuel_delivery_costTRY_per_waterLitre, data =.)
```


Another scatter plot 'Fuel cost for delivery of water according to delivery distance with all types of KI' is to see the correction of the fuel cost of water delivery and delivery distance for all types of KI.

![paste to excel](https://github.com/tinatmyiu/casestudy/blob/5de2e211dd8aaac3ec7e72be0d30f939a1b09996/Fuel%20cost%20for%20delivery%20of%20water%20according%20to%20delivery%20distance%20with%20all%20types%20of%20KI.png)

```r
#Data wrangling for all ki_type
fuel_water_cost_all <- select(main_data, ki_type, cost_fuel_delivery1, delivery_volume, delivery_distance) %>%
  mutate(delivery_volume_litre = delivery_volume*158.987294928) %>%
  mutate(fuel_delivery_costTRY_per_waterLitre = cost_fuel_delivery1/delivery_volume_litre) %>%
  na.omit(fuel_delivery_costTRY_per_waterLitre)

#Data visualization for all ki_type
ggplot(fuel_water_cost_all, aes(x=delivery_distance, y=fuel_delivery_costTRY_per_waterLitre, color= ki_type)) + 
  geom_point(size=4) +
  labs(title="Fuel cost for delivery of water according to delivery distance with all types of KI",
       x="Delivery distance (km)", y = "Fuel cost per water (TRY/Litre)")
```

## Conclusion and recommendations
The mean fuel cost per litre of water is TRY 0.007010478/Litre for delivery of private water trucking activities in NWS. As the p-vaule of 'delivery_distance' and 'fuel_delivery_costTRY_per_waterLitre' is 0.6764, which is bigger than 0.05. Correlation coefficient is -0.04707941, which is close to 0. They show no certainty in results and no correlation. The fuel cost for delivering water is independent of delivery distance. The fuel cost of private water trucking activities does not show significant differecnce with other water trucking activies. 

A more comprehensive evauation can be made if other costs, such as car fees and registration fees, are also included in data analysis. However, more questions will be needed. For example, how much water was delivered per month? To have a fair comparison, it is needed to align different variables to the same length of time.
