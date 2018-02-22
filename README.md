

```python
# Import Dependencies
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib qt5    
# qt5 = popup window
```


```python
city_raw_data = "raw_data/city_data.csv"
ride_raw_data = "raw_data/ride_data.csv"

city_data = pd.read_csv(city_raw_data, low_memory=False)
ride_data = pd.read_csv(ride_raw_data, low_memory=False)
# city_data.columns
# city_data.head()
# ride_data.columns
# ride_data.head()

city_complete = pd.merge(city_data, ride_data, how="left", on=["city","city"])
# city_complete.columns
city_complete.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>driver_count</th>
      <th>type</th>
      <th>date</th>
      <th>fare</th>
      <th>ride_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kelseyland</td>
      <td>63</td>
      <td>Urban</td>
      <td>2016-08-19 04:27:52</td>
      <td>5.51</td>
      <td>6246006544795</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kelseyland</td>
      <td>63</td>
      <td>Urban</td>
      <td>2016-04-17 06:59:50</td>
      <td>5.54</td>
      <td>7466473222333</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Kelseyland</td>
      <td>63</td>
      <td>Urban</td>
      <td>2016-05-04 15:06:07</td>
      <td>30.54</td>
      <td>2140501382736</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Kelseyland</td>
      <td>63</td>
      <td>Urban</td>
      <td>2016-01-25 20:44:56</td>
      <td>12.08</td>
      <td>1896987891309</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Kelseyland</td>
      <td>63</td>
      <td>Urban</td>
      <td>2016-08-09 18:19:47</td>
      <td>17.91</td>
      <td>8784212854829</td>
    </tr>
  </tbody>
</table>
</div>




```python
# print(city_data.columns)
# print(ride_data.columns)
# print(city_complete.columns)
```


```python
total_cities = len(city_complete.city.unique())
average_fare_all = city_complete.fare.mean()
print("total cities: "+ str(total_cities) + "\navg fare for all: " + str(average_fare_all))
```

    total cities: 125
    avg fare for all: 26.86710427918569



```python
average_fares = city_complete.groupby(["city"]).mean()["fare"]
# average_fares.head()
rides_per_city  = city_complete.groupby(["city"]).count()["fare"]
# rides_per_city.head()
drivers_per_city = city_complete.groupby("city").sum()["driver_count"]
# drivers_per_city.head()
# city_type = city_complete.groupby("city")["type"].value_counts()
sum_fares = city_complete.groupby(["city"]).sum()["fare"]
city_type = city_complete.groupby("city")["type"]
# city_type = city_complete.groupby("city").max()["type"]
city_type2 = city_complete.groupby("city")["type"].value_counts()



city_summary = pd.DataFrame({"Drivers" : drivers_per_city,
                             "Rides" : rides_per_city,
                             "Revenue" : sum_fares,
                             "Average Fares" : average_fares
                            })
#
# rearrange your data cause your stupid data is causing massive issues
# and
# city_type = city_complete.groupby("city").max()["type"]
# is not an elegant way of doing business when it comes to non-numbers
#
WhyAreThereTwo = city_data.set_index(["type"])
WhyAreThereTwo.index.value_counts()
# WhyAreThereTwo.head()
WhyAreThereTwo = WhyAreThereTwo["city"].drop_duplicates()
WhyAreThereTwo = WhyAreThereTwo.reset_index().set_index(["city"])
city_summary2 = pd.concat([city_summary,WhyAreThereTwo["type"]],axis=1)
city_summary2 = city_summary2.rename(columns={'type':'Type'}) 
# city_summary2

```


```python
# # graph the old way
# x = rides_per_city
# y = average_fares
# z = drivers_per_city
# plt.scatter(x, y, s=z*2, alpha=0.3, linewidth=4)

# seaborn graph:
sns.lmplot( x="Rides", y="Average Fares", data=city_summary2, fit_reg=False, hue='Type', legend=False, palette=dict(Rural="#FFD700", Suburban="#87CEFA", Urban="#F08080"), scatter_kws={"alpha":0.6,"s":.25*drivers_per_city, "linewidth":2})
# plt.ylim(0, None)
# plt.xlim(0, None) 
plt.legend(loc='upper right')

plt.xlabel("Number of Rides (per City)")
plt.ylabel("Average Fare (in USD)")
plt.title("Pyber Ride Sharing Data")

# Annotate with to make it look like the example in HW that nick made
plt.annotate(
'Note: bubble size corresponds\n with number of drivers\n (larger bubble= more drivers)', xy=(32, 30), xytext=(32, 40),
)


plt.show()



```


```python
# pie data:

city_by_type_df = city_summary2.set_index(["Type"])
city_by_type_df

sum_fares_by_type = city_by_type_df.groupby(["Type"]).sum()["Revenue"]
# sum_fares_by_type
sum_fares_by_type_total = sum_fares_by_type.sum()
sum_rides_by_type  = city_by_type_df.groupby(["Type"]).sum()["Rides"]
# sum_rides_by_type
sum_rides_by_type_total = sum_rides_by_type.sum()
sum_drivers_by_type = city_by_type_df.groupby("Type").sum()["Drivers"]
# sum_drivers_by_type
sum_drivers_by_type_total = sum_drivers_by_type.sum()

# rural:
percent_fares_rural = (sum_fares_by_type["Rural"]/sum_fares_by_type_total)*100
# percent_fares_rural
percent_rides_rural = (sum_rides_by_type["Rural"]/sum_rides_by_type_total)*100
# percent_rides_rural
percent_drivers_rural = (sum_drivers_by_type["Rural"]/sum_drivers_by_type_total)*100
# percent_drivers_rural

# suburban:
percent_fares_suburban = (sum_fares_by_type["Suburban"]/sum_fares_by_type_total)*100
# percent_fares_suburban
percent_rides_suburban = (sum_rides_by_type["Suburban"]/sum_rides_by_type_total)*100
# percent_rides_suburban
percent_drivers_suburban = (sum_drivers_by_type["Suburban"]/sum_drivers_by_type_total)*100
# percent_drivers_suburban

# urban:
percent_fares_urban = (sum_fares_by_type["Urban"]/sum_fares_by_type_total)*100
# percent_fares_urban
percent_rides_urban = (sum_rides_by_type["Urban"]/sum_rides_by_type_total)*100
# percent_rides_urban
percent_drivers_urban = (sum_drivers_by_type["Urban"]/sum_drivers_by_type_total)*100
# percent_drivers_urban
# "{:,.2f}%".format(percent_drivers_urban)

pie_summary = pd.DataFrame({"Fares" : sum_fares_by_type,
                            "Rides" : sum_rides_by_type,
                            "Drivers" : sum_drivers_by_type,
                            "% Drivers" : (sum_drivers_by_type / sum_drivers_by_type_total)*100,
                            "% Fares" : (sum_fares_by_type / sum_fares_by_type_total)*100,
                            "% Rides" : (sum_rides_by_type / sum_rides_by_type_total)*100
                            },
                          index = ["Urban", "Suburban", "Rural"])
# pie_summary["% Drivers"]
pie_summary["% Drivers"] = pie_summary["% Drivers"].map("{:,.2f}%".format)
pie_summary["% Fares"] = pie_summary["% Fares"].map("{:,.2f}%".format)
pie_summary["% Rides"] = pie_summary["% Rides"].map("{:,.2f}%".format)
# pie_summary

```


```python
# % of Total Fares by City Type
# Pie chart
labels = pie_summary.index
sizes = pie_summary["Fares"]
explode = (.15, 0, 0)  

fig1, ax1 = plt.subplots()
ax1.pie(sizes, explode=explode, labels=labels, autopct='%1.1f%%',
        shadow=True, startangle=275, colors=["#F08080", "#87CEFA","#FFD700"])
ax1.axis('equal')

plt.title("Percent of Total Fares by City Type")
plt.legend(loc='upper left')
plt.show()
```


```python


# look what i can do
group_names= pie_summary.index
group_size= pie_summary["Fares"]
subgroup_names=city_summary2.index
subgroup_size=city_summary2["Revenue"]
 
# Interior colors
a, b, c=[plt.cm.Blues, plt.cm.Reds, plt.cm.Greens]
 
# First Ring (outside)
fig, ax = plt.subplots()
ax.axis('equal')
mypie, _ = ax.pie(group_size, radius=1.3, startangle=275, labels=group_names, colors=["#F08080", "#87CEFA","#FFD700"] )
plt.setp( mypie, width=0.3, edgecolor='white')
 
# Second Ring (Inside)
mypie2, _ = ax.pie(subgroup_size, radius=1.3-0.3, startangle=275, labels=subgroup_names, labeldistance=1, colors=[a(0.5), a(0.4), a(0.3), b(0.5), b(0.4), c(0.6), c(0.5), c(0.4), c(0.3), c(0.2)])
plt.setp( mypie2, width=0.4, edgecolor='white')
plt.margins(0,0)
 
# show it
plt.title("Fare Revenue by Type and Fare Revenue per City therein")
plt.show()

```


```python
# % of Total Rides by City Type

# Pie chart
labels = pie_summary.index
sizes = pie_summary["Rides"]
explode = (.15, 0, 0)  

fig1, ax1 = plt.subplots()
ax1.pie(sizes, explode=explode, labels=labels, autopct='%1.1f%%',
        shadow=True, startangle=275, colors=["#F08080", "#87CEFA","#FFD700"])
ax1.axis('equal')

plt.title("Percent of Total Rides by City Type")
plt.legend(loc='upper left')
plt.show()
```


```python

# look what i can do
group_names= pie_summary.index
group_size= pie_summary["Rides"]
subgroup_names=city_summary2.index
subgroup_size=city_summary2["Rides"]
 
# Interior colors
a, b, c=[plt.cm.Blues, plt.cm.Reds, plt.cm.Greens]
 
# First Ring (outside)
fig, ax = plt.subplots()
ax.axis('equal')
mypie, _ = ax.pie(group_size, radius=1.3, startangle=275, labels=group_names, colors=["#F08080", "#87CEFA","#FFD700"] )
plt.setp( mypie, width=0.3, edgecolor='white')
 
# Second Ring (Inside)
mypie2, _ = ax.pie(subgroup_size, radius=1.3-0.3, startangle=275, labels=subgroup_names, labeldistance=1, colors=[a(0.5), a(0.4), a(0.3), b(0.5), b(0.4), c(0.6), c(0.5), c(0.4), c(0.3), c(0.2)])
plt.setp( mypie2, width=0.4, edgecolor='white')
plt.margins(0,0)
 
# show it
plt.title("Rides by Type and Rides per City therein")
plt.show()
```


```python
# % of Total Drivers by City Type
# Pie chart
labels = pie_summary.index
sizes = pie_summary["Drivers"]
explode = (.15, 0, 0)  

fig1, ax1 = plt.subplots()
ax1.pie(sizes, explode=explode, labels=labels, autopct='%1.1f%%',
        shadow=True, startangle=275, colors=["#F08080", "#87CEFA","#FFD700"])
ax1.axis('equal')

plt.title("Percent of Total Drivers by City Type")
plt.legend(loc='upper left')
plt.show()
```


```python
# look what i can do
group_names= pie_summary.index
group_size= pie_summary["Drivers"]
subgroup_names=city_summary2.index
subgroup_size=city_summary2["Drivers"]
 
# Interior colors
a, b, c=[plt.cm.Blues, plt.cm.Reds, plt.cm.Greens]
 
# First Ring (outside)
fig, ax = plt.subplots()
ax.axis('equal')
mypie, _ = ax.pie(group_size, radius=1.3, startangle=275, labels=group_names, colors=["#F08080", "#87CEFA","#FFD700"] )
plt.setp( mypie, width=0.3, edgecolor='white')
 
# Second Ring (Inside)
mypie2, _ = ax.pie(subgroup_size, radius=1.3-0.3, startangle=275, labels=subgroup_names, labeldistance=1, colors=[a(0.5), a(0.4), a(0.3), b(0.5), b(0.4), c(0.6), c(0.5), c(0.4), c(0.3), c(0.2)])
plt.setp( mypie2, width=0.4, edgecolor='white')
plt.margins(0,0)
 
# show it
plt.title("Number of Drivers by Type and Number of Drivers per City therein")
plt.show()


```

Some observations:

1. Rural cities have the least number of rides but the highest average fare cost
2. Urban cities have the highest number of rides but the lowest average fare cost
3. Pyber is most used in Urban cities
