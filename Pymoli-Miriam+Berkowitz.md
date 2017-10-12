
# Heroes Of Pymoli Data Analysis<br>
### Observed Trends:<br>
1) Male players outnumber female by almost 5:1.<br>
2) About 40 percent of players are between the ages of 20 and 24. 75 percent fall between ages 15 and 29.<br> 
3) While 20-24 year olds made the most purchases, those over age 40 spend more per capita than any other group.<br>
4) The "Hatred" item was both a best seller and made the most revenue. 


```python
#Dependencies
import pandas as pd
```


```python
#read json file of dictionary items
#json_path = "raw_data/purchase_data.json"
#p_data_df = pd.read_json(json_path)

#read CSV file
path = "raw_data/purchase_data_3.csv"
p_data_df = pd.read_csv(path)

p_data_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase ID</th>
      <th>SN</th>
      <th>Age</th>
      <th>Gender</th>
      <th>Item ID</th>
      <th>Item Name</th>
      <th>Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Lirtjaskan85</td>
      <td>24</td>
      <td>Female</td>
      <td>69</td>
      <td>Frenzy, Defender of the Harvest</td>
      <td>4.82</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Chanjask65</td>
      <td>12</td>
      <td>Female</td>
      <td>75</td>
      <td>Brutality Ivory Warmace</td>
      <td>4.12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Aerithllora36</td>
      <td>21</td>
      <td>Female</td>
      <td>114</td>
      <td>Yearning Mageblade</td>
      <td>2.67</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Aeralria27</td>
      <td>24</td>
      <td>Male</td>
      <td>130</td>
      <td>Alpha</td>
      <td>4.53</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Haisrisuir60</td>
      <td>28</td>
      <td>Male</td>
      <td>9</td>
      <td>Thorn, Conqueror of the Corrupted</td>
      <td>4.60</td>
    </tr>
  </tbody>
</table>
</div>



### Player Count


```python
unique_player_count_s = pd.Series(['Total Players',len(p_data_df['SN'].unique())])

#keep track of number of unique players for later calculations
unique_players = len(p_data_df['SN'].unique())

#output number of unique players
unique_player_count_s

```




    0    Total Players
    1              581
    dtype: object



### Purchasing Analysis (Total)


```python
#total number of purchases
total_purchases = p_data_df['SN'].count()
stats = p_data_df.describe()

price_mean = stats.iloc[1,3]
price_mean
item_list = p_data_df['Item ID'].value_counts()
unique_item_count = item_list.count()
total_revenue = p_data_df['Price'].sum()

```


```python
purch_analysis = pd.DataFrame({'Number of Unique Items': [unique_item_count],
                              'Average Purchase Price': [price_mean],
                             'Total Number of Purchases':[total_purchases],
                              'Total Revenue':[total_revenue]})
#put the columns in the right order
purch_analysis = purch_analysis[["Number of Unique Items",
                              "Average Purchase Price",
                                 "Total Number of Purchases",
                                 "Total Revenue"]]
#add the formatting

purch_analysis['Average Purchase Price'] = purch_analysis['Average Purchase Price'].map('${:,.2f}'.format)
purch_analysis['Total Revenue'] = purch_analysis['Total Revenue'].map('${:,.2f}'.format)
purch_analysis


```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number of Unique Items</th>
      <th>Average Purchase Price</th>
      <th>Total Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>181</td>
      <td>$3.03</td>
      <td>780</td>
      <td>$2,365.17</td>
    </tr>
  </tbody>
</table>
</div>



### Gender Demographics


```python
#need to first get a table of unique players, and then group by gender
unique_players_df = p_data_df[['Gender','SN','Age']]
#remove duplicate SN's
unique_players_df = unique_players_df.drop_duplicates(subset=['SN'])
```


```python
gender_grouped = unique_players_df.groupby(['Gender'])
gender_demo = pd.DataFrame(gender_grouped['Gender'].count())
gender_demo = gender_demo.rename(columns={'Gender':'Total Count'})
gender_demo = gender_demo.reset_index()

#compute the percentage
gender_demo['Percentage of Players'] =   round((gender_demo["Total Count"]/unique_players) * 100 ,2)
gender_demo.set_index('Gender',inplace=True)
gender_demo


```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Count</th>
      <th>Percentage of Players</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>99</td>
      <td>17.04</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>475</td>
      <td>81.76</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>7</td>
      <td>1.20</td>
    </tr>
  </tbody>
</table>
</div>



### Purchasing Analysis (Gender)


```python
#now go back to the original data to get the price info
gender_grouped = p_data_df.groupby(['Gender'])
gender_grouped['Price'].mean()

```




    Gender
    Female                   3.044615
    Male                     3.030389
    Other / Non-Disclosed    2.982500
    Name: Price, dtype: float64




```python
#create data frame for purchasing analysis, and put in the count of the purchases (by gender)
purch_gender_df = pd.DataFrame(gender_grouped['Price'].count())
purch_gender_df = purch_gender_df.rename(columns={"Price":"Purchase Count"})

#add the mean and the sum
purch_gender_df['Average Purchase Price'] = gender_grouped['Price'].mean()
purch_gender_df['Total Purchase Value'] = gender_grouped['Price'].sum()
purch_gender_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>130</td>
      <td>3.044615</td>
      <td>395.80</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>642</td>
      <td>3.030389</td>
      <td>1945.51</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>8</td>
      <td>2.982500</td>
      <td>23.86</td>
    </tr>
  </tbody>
</table>
</div>




```python
#add the Normalized Totals 

#Try this for normalizing the average:
#Total Purchase Value / number of unique people in the category

purch_gender_df['Normalized Averages']= purch_gender_df['Total Purchase Value'] / gender_demo['Total Count']

#add formatting
purch_gender_df['Average Purchase Price'] = purch_gender_df['Average Purchase Price'].map('${:,.2f}'.format)
purch_gender_df['Total Purchase Value'] = purch_gender_df['Total Purchase Value'].map('${:,.2f}'.format)
purch_gender_df['Normalized Averages']= purch_gender_df['Normalized Averages'].map('${:,.2f}'.format)

purch_gender_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Averages</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>130</td>
      <td>$3.04</td>
      <td>$395.80</td>
      <td>$4.00</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>642</td>
      <td>$3.03</td>
      <td>$1,945.51</td>
      <td>$4.10</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>8</td>
      <td>$2.98</td>
      <td>$23.86</td>
      <td>$3.41</td>
    </tr>
  </tbody>
</table>
</div>



### Age Demographics


```python
#Break into bins of 4 years (percentage of players, total count)
# and then in separate output: (purchase count, avg purch price, tot purch value, normalized)
bins = [0,10,14,19,24,29,34,39,100]
group_names = ['<10','10-14','15-19','20-24','25-29','30-34','35-39','40+']
index_names = [1,2,3,4,5,6,7,8]

#add the bin to the list of UNIQUE players
unique_players_df['age group'] = pd.cut(unique_players_df['Age'], bins, labels=index_names)
#group by the bins
age_groups = unique_players_df.groupby("age group")
#create output with total
age_demog = pd.DataFrame(age_groups['Age'].count())

```


```python
#rename group to Total Count, change the age to the actual group names and add a column for percentage
age_demog = age_demog.rename(columns={"Age":"Total Count"})

age_demog.reset_index(inplace=True)
age_demog['age group'] = age_demog['age group'].replace(index_names, group_names)
age_demog['Percentage of Players'] = round((age_demog['Total Count']/unique_players) * 100,2)

#set the index to be the age group
age_demog.set_index('age group',inplace=True)
age_demog
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Count</th>
      <th>Percentage of Players</th>
    </tr>
    <tr>
      <th>age group</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>28</td>
      <td>4.82</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>19</td>
      <td>3.27</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>103</td>
      <td>17.73</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>230</td>
      <td>39.59</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>105</td>
      <td>18.07</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>47</td>
      <td>8.09</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>41</td>
      <td>7.06</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>8</td>
      <td>1.38</td>
    </tr>
  </tbody>
</table>
</div>



### Purchasing Analysis (Age)


```python
#add the bins to the original data read
p_data_df['age group'] = pd.cut(p_data_df['Age'], bins, labels=index_names)
#group by the bins
age_groups = p_data_df.groupby("age group")

```


```python
#create data frame 
purch_analysis = pd.DataFrame(age_groups['Item ID'].count())

#add column with avg purchase price
purch_analysis['Average Purchase Price'] = age_groups['Price'].mean()

#add total purchase value for each age group
purch_analysis['Total Purchase Value'] = age_groups['Price'].sum()

#rename column
purch_analysis = purch_analysis.rename(columns={"Item ID":"Purchase Count"})

#set the index to be the age group, and put in the group names (so they will be in the correct order)
purch_analysis.reset_index(inplace=True)
purch_analysis['age group'] = purch_analysis['age group'].replace(index_names, group_names)

purch_analysis.set_index('age group',inplace=True)
purch_analysis
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>age group</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>39</td>
      <td>2.922308</td>
      <td>113.97</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>29</td>
      <td>2.979655</td>
      <td>86.41</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>132</td>
      <td>3.080455</td>
      <td>406.62</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>316</td>
      <td>3.008196</td>
      <td>950.59</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>137</td>
      <td>3.045328</td>
      <td>417.21</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>62</td>
      <td>2.925806</td>
      <td>181.40</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>54</td>
      <td>3.175926</td>
      <td>171.50</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>11</td>
      <td>3.406364</td>
      <td>37.47</td>
    </tr>
  </tbody>
</table>
</div>




```python
#add the normalized averages
#Try this for normalizing the average:
#Total Purchase Value / number of unique people in the category
#    this appears to be average spending per capita
purch_analysis['Normalized Averages']= purch_analysis['Total Purchase Value'] / age_demog['Total Count']

#add formatting
purch_analysis['Average Purchase Price'] = purch_analysis['Average Purchase Price'].map('${:,.2f}'.format)
purch_analysis['Total Purchase Value'] = purch_analysis['Total Purchase Value'].map('${:,.2f}'.format)
purch_analysis['Normalized Averages']= purch_analysis['Normalized Averages'].map('${:,.2f}'.format)

purch_analysis
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Averages</th>
    </tr>
    <tr>
      <th>age group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>39</td>
      <td>$2.92</td>
      <td>$113.97</td>
      <td>$4.07</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>29</td>
      <td>$2.98</td>
      <td>$86.41</td>
      <td>$4.55</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>132</td>
      <td>$3.08</td>
      <td>$406.62</td>
      <td>$3.95</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>316</td>
      <td>$3.01</td>
      <td>$950.59</td>
      <td>$4.13</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>137</td>
      <td>$3.05</td>
      <td>$417.21</td>
      <td>$3.97</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>62</td>
      <td>$2.93</td>
      <td>$181.40</td>
      <td>$3.86</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>54</td>
      <td>$3.18</td>
      <td>$171.50</td>
      <td>$4.18</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>11</td>
      <td>$3.41</td>
      <td>$37.47</td>
      <td>$4.68</td>
    </tr>
  </tbody>
</table>
</div>



### Top Spenders


```python
#group by SN, and get avg price, sum of price and count of price
spenders = p_data_df.groupby(['SN'])
spenders.head()

total_spenders = pd.DataFrame(spenders['Price'].count())
total_spenders["Purchase Count"] = spenders['Price'].count()
total_spenders["Average Purchase Price"] = spenders['Price'].mean()
total_spenders["Total Purchase Value"] = spenders['Price'].sum()

#delete extra column
del total_spenders['Price']

#sort by total, descending
top_spenders = total_spenders.sort_values("Total Purchase Value", ascending=False)

#format
top_spenders['Average Purchase Price'] = top_spenders['Average Purchase Price'].map('${:,.2f}'.format)
top_spenders['Total Purchase Value'] = top_spenders['Total Purchase Value'].map('${:,.2f}'.format)

#display the top 5 spenders
top_spenders.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Chamalo71</th>
      <td>4</td>
      <td>$3.36</td>
      <td>$13.45</td>
    </tr>
    <tr>
      <th>Strithenu87</th>
      <td>4</td>
      <td>$3.21</td>
      <td>$12.83</td>
    </tr>
    <tr>
      <th>Mindosia50</th>
      <td>3</td>
      <td>$4.00</td>
      <td>$12.01</td>
    </tr>
    <tr>
      <th>Aeralria27</th>
      <td>3</td>
      <td>$3.79</td>
      <td>$11.38</td>
    </tr>
    <tr>
      <th>Eudai71</th>
      <td>3</td>
      <td>$3.79</td>
      <td>$11.37</td>
    </tr>
  </tbody>
</table>
</div>



### Most Popular Items


```python
#group by Item ID&Name, and get count, price and total value (sum)
items = p_data_df.groupby(['Item ID','Item Name','Price'])
items.head()
total_items = pd.DataFrame(items['Item Name'].count())

total_items['Total Purchase Value'] = items['Price'].sum()

total_items = total_items.rename(columns={'Item Name':'Purchase Count'})

total_items.reset_index(inplace=True)


#put the columns in the right order and change the index to Item ID
total_items = total_items[['Item ID','Item Name','Purchase Count','Price','Total Purchase Value']]
total_items.set_index('Item ID',inplace=True)
total_items = total_items.rename(columns={'Price':'Item Price'})

#sort to get most popular
sorted_by_purchase = total_items.sort_values("Purchase Count", ascending=False)

#sort to get most profitable (for next section), before formatting
sorted_by_revenue = total_items.sort_values("Total Purchase Value", ascending=False)

# add the formatting
sorted_by_purchase['Item Price'] = sorted_by_purchase['Item Price'].map('${:,.2f}'.format)
sorted_by_purchase['Total Purchase Value'] = sorted_by_purchase['Total Purchase Value'].map('${:,.2f}'.format)

#display the top 5
sorted_by_purchase.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Item Name</th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>52</th>
      <td>Hatred</td>
      <td>11</td>
      <td>$4.59</td>
      <td>$50.49</td>
    </tr>
    <tr>
      <th>174</th>
      <td>Primitive Blade</td>
      <td>9</td>
      <td>$1.39</td>
      <td>$12.51</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Misery's End</td>
      <td>9</td>
      <td>$4.90</td>
      <td>$44.10</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Celeste</td>
      <td>8</td>
      <td>$2.07</td>
      <td>$16.56</td>
    </tr>
    <tr>
      <th>143</th>
      <td>Frenzied Scimitar</td>
      <td>8</td>
      <td>$3.35</td>
      <td>$26.80</td>
    </tr>
  </tbody>
</table>
</div>



### Most Profitable Items


```python
#sort to get "most profitable"
# This is really the items with the highest revenue. Don't really know if they are most profitable 
#     because we don't know the expenses associated with them. 

sorted_by_revenue = total_items.sort_values("Total Purchase Value", ascending=False)

# add the formatting
sorted_by_revenue['Item Price'] = sorted_by_revenue['Item Price'].map('${:,.2f}'.format)
sorted_by_revenue['Total Purchase Value'] = sorted_by_revenue['Total Purchase Value'].map('${:,.2f}'.format)

#display the top 5
sorted_by_revenue.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Item Name</th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>52</th>
      <td>Hatred</td>
      <td>11</td>
      <td>$4.59</td>
      <td>$50.49</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Misery's End</td>
      <td>9</td>
      <td>$4.90</td>
      <td>$44.10</td>
    </tr>
    <tr>
      <th>120</th>
      <td>Agatha</td>
      <td>8</td>
      <td>$4.93</td>
      <td>$39.44</td>
    </tr>
    <tr>
      <th>93</th>
      <td>Apocalyptic Battlescythe</td>
      <td>8</td>
      <td>$4.85</td>
      <td>$38.80</td>
    </tr>
    <tr>
      <th>49</th>
      <td>The Oculus, Token of Lost Worlds</td>
      <td>8</td>
      <td>$4.61</td>
      <td>$36.88</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
