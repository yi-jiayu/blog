---
title: "Analysing HDB Resale Flat Transactions With VisiData"
date: 2021-12-13T13:38:45+08:00
image: scatter.png
---

[VisiData](https://www.visidata.org/) is a quick way to run simple queries on CSV data without having to load the data into a database or use R, Python or Excel.

We can find a [resale flat prices dataset](https://data.gov.sg/dataset/resale-flat-prices) on Data.gov.sg. It contains transaction data over multiple time periods in CSV format:

```
$ wget --content-disposition https://data.gov.sg/dataset/7a339d20-3c57-4b11-a695-9348adfd7614/download
$ unzip -d resale-flat-prices resale-flat-prices.zip
$ cd resale-flat-prices
$ exa
metadata-resale-flat-prices.txt
resale-flat-prices-based-on-approval-date-1990-1999.csv
resale-flat-prices-based-on-approval-date-2000-feb-2012.csv
resale-flat-prices-based-on-registration-date-from-jan-2015-to-dec-2016.csv
resale-flat-prices-based-on-registration-date-from-jan-2017-onwards.csv
resale-flat-prices-based-on-registration-date-from-mar-2012-to-dec-2014.csv
```

We'll focus on the most recent data contained in `resale-flat-prices-based-on-registration-date-from-jan-2017-onwards.csv`. To open the file in VisiData:

```
vd resale-flat-prices-based-on-registration-date-from-jan-2017-onwards.csv
```

# Median price by town and flat type

1. Use the left and right arrow keys or `H` and `L` to navigate between
   columns.
2. Navigate to the `town` column and press `!` to set it as the row label.
3. Navigate to the `resale_price` column and press `#` to mark it as a numeric
   column.
4. With the `resale_price` column still selected, press `+` to choose an
   aggregation, then type `median` and press enter.
5. Navigate to the `flat_type` column and press `Shift+W` to generate a pivot
   table with it as the column label.
6. In the generated pivot table, press `[` or `]` with a column selected to
   sort by that column.

The end result and the process of getting there:

<script id="asciicast-455754" src="https://asciinema.org/a/455754.js"></script>

# Price by floor area and flat type

VisiData can also generate colourful scatter plots:

{{< figure src="scatter.png" class="nooutline" >}}

Starting from a fresh instance of our data in VisiData:

1. Navigate to the `floor_area_sqm` column and press `#` to mark it as
   numerical, then `!` to set it as a key column.
2. Navigate to the `flat_type` column and press `!` to mark it as a second key
   column.
3. Navigate to the `resale_price` column and press `#` to mark it as numerical,
   then press `.` to generate the scatter plot.

# Median price by floor and flat type

We can use derived columns when the raw data in a column is not in an appropriate format. For example, the `storey_range` column contains values like `01 TO 03` or `10 TO 12`, which is not properly numeric, so we cannot use it directly. Instead, we can create a new column derived from it containing just the lower bound of the range.

Starting from a fresh instance of our data:

1. Press `=` to create a new column, then type `storey_range[:2]` and press
   enter. This will be evaluated as a Python expression, creating a new column
   containing the first two characters of the `storey_range` column.
2. Navigate to the new `storey_range[:2]` column and press `#` to mark it as a
   numeric column, then press `!` to use it as the row label.
3. Navigate to the `resale_price` column and press `#` to mark it as numerical,
   then press `+` to add an aggregation and type in `median` and press enter.
4. Navigate to the `flat_type` column and press `Shift+W` to generate a pivot
   table with it as the column label.
5. In the generated pivot table, press `[` or `]` on any column to sort by that
   column, or press `.` to view a scatter plot of just that column.

Animated example:

<script id="asciicast-uTdrjaXd9oUNIeHYOmzeJm76y" src="https://asciinema.org/a/uTdrjaXd9oUNIeHYOmzeJm76y.js" async></script>

Scatter plot:

{{< figure src="price-vs-storey.png" class="nooutline" >}}
