# Centraal Bureau voor de Statistiek api

Currently I'm working on a few small projects for a portfolio that shows my knowledge and skill in Java and Spring. The first mini-project I'm working on is a one-page website with a single search field. You can type the name of a dutch town (it has an autofill similar to buienradar.nl) and get some statistics on demographics. Initially I wanted to add some data on available shops, but accessing the internal api's of retailers (Kruidvat, Etos, Trekpleister) via a Java Spring application was more of a pain than I expected. Those endpoints are protected and can only easily be accessed via a browser or apps like Postman and Boomerang.

## Pdok and CBS

[PDOK](https://www.pdok.nl/) is a platform providing dutch geodata. I used their api to retrieve a list of dutch towns and it worked fine. A link like [this](https://api.pdok.nl/bzk/locatieserver/search/v3_1/suggest?q=type:woonplaats) gives you the complete list, [this one](https://api.pdok.nl/bzk/locatieserver/search/v3_1/suggest?q=type:gemeente) provides all municipalities and if you want details of towns, you can do it with a link like [this](https://api.pdok.nl/bzk/locatieserver/search/v3_1/lookup?id=wpl-421d72725f03866e9aee43e333794774).

CBS (Centraal Bureau voor de Statistiek) is a different story. They have an [api](https://opendata.cbs.nl/ODataApi/) where you can do extensive search in a ton of tables, but probably due to the sheer amount of data, the large number of topics and dimensions, and the long period over which the data was collected, it is way harder to create url's that will return any data at all, let alone the data you were looking for.

## Decrypting the CBS Open Data portal

I created a set of small chapters that help to understand the CBS tables and the working of api. Without understanding how the tables work and how the variables are named and used it is not possible to generate proper url's, so everything below is required reading.

### 1 - Find the right table

Go to the [CBS website](https://opendata.cbs.nl/statline/portal.html?_la=nl&_catalog=CBS). You can also go [here](https://opendata.cbs.nl/statline/#/CBS/nl/navigatieScherm/thema), this one has better navigation. You can search for a table with a description that matches your search request. [This link](https://opendata.cbs.nl/ODataApi/) provides a full listing as well, without further descriptions.

### 2 - Check the table preview 

The preview that you will see shows just one of the many ways in which data can be ordered and subselections can be made. You can drag the variables around (I think there is a maximum of seven) and you can click the funnel symbols to make subselections within a variable (or call it a topic or a dimension). What you learn here is whether the table is able to generate the info you need. For example, if you want a fine grained geographical level (municipality or district ('wijk')), you should be able to select municipalities or 'wijken' when you click a variable related to geography.

### 3 - Create the base url for the table api

The url of the table page contains a code, five digits followed by either 'NED' or 'ENG. Create a new url like this (the last path segment points to a specific table):

```
https://opendata.cbs.nl/ODataApi/OData/83642NED/
```

### 4 - Figure out variable names

This base url provides all the possible basic links to the data, dimensions and metadata of the table. This is what it looks for table 83642NED:

```
{
  "odata.metadata": "https://opendata.cbs.nl/ODataApi/OData/83642NED/$metadata",
  "value": [
    {
      "name": "TableInfos",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/TableInfos"
    },
    {
      "name": "UntypedDataSet",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/UntypedDataSet"
    },
    {
      "name": "TypedDataSet",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/TypedDataSet"
    },
    {
      "name": "DataProperties",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/DataProperties"
    },
    {
      "name": "CategoryGroups",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/CategoryGroups"
    },
    {
      "name": "GemeentelijkeHeffingenVanaf2017",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/GemeentelijkeHeffingenVanaf2017"
    },
    {
      "name": "RegioS",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/RegioS"
    },
    {
      "name": "Perioden",
      "url": "https://opendata.cbs.nl/ODataApi/OData/83642NED/Perioden"
    }
  ]
}
```

Any of these links is valid and tells you more. The `/UntypedDataSet` and `/TypedDataSet` are the path segments that you will need for inquiry into the data, the others are for metadata and dimensions (see next subchapters). `/UntypedDataSet` and `/TypedDataSet` often return too much data rows and thus require refinement (more on that later). 

A useful link, sort of hack, is this one:

```
https://opendata.cbs.nl/ODataApi/OData/83642NED/TypedDataSet?$top=1
```

It provides a list of the variable names you can use in your queries.

### 4 - Understand Dimension and Topic

If you visit the `/DataProperties` endpoint you find a list of values that can be of the following odata types:

- Dimension
- Topic
- TopicGroup
- GeoDimension
- GeoDetail
- TimeDimension

The important thing here is to understand the difference between Dimension and Topic. Note that GeoDimension, GeoDetail and TimeDimension are all specialized and standardized dimension types, and that TopicGroup is a sort of non-consequential dummy object that groups similar Topics.

The difference between Dimension and Topic is that the latter refers to the things that are measured, resulting in the numbers, codes or maybe boolean values that you find in the tables. A topic can be things like 'number of bankruptcies' or 'average income'. 

Dimension is the thing that specifies which items should be counted and which should not be counted for a given topic. For example, if you want to limit the measurement of average income to a certain age group, then average income is the topic and age group is the dimension. This explains why GeoDetail, GeoDimension and TimeDimension are considered dimensions and not topics, as we use them to restrict the number of things to measure to a specific period or geographical area. 

### 5 - The structure of a dimension

It is worthwile to note that while all topics used in a table can be found in `/DataProperties` or via `/TypedDataSet?$top=1`, the dimensions are only found by their main description. For example, table 86052NED has a dimension 'Opleidingsniveau' that shows up like this in /DataProperties:

```
    {
      "odata.type": "Cbs.OData.Dimension",
      "ID": 0,
      "Position": 0,
      "ParentID": null,
      "Type": "Dimension",
      "Key": "Opleidingsniveau",
      "Title": "Opleidingsniveau",
      "Description": ""
    }
```

Nowhere in /DataProperties you can find that opleidingsniveau has the following options:

- 1 Basisonderwijs, vmbo, mbo1
- 2 Havo, vwo, mbo2-4
- 3 Hbo, wo

But if you visit the following endpoint:

```
https://opendata.cbs.nl/ODataApi/OData/86052NED/Opleidingsniveau
```

You see how the dimension is structured:

```
{
  "odata.metadata": "https://opendata.cbs.nl/ODataApi/OData/86052NED/$metadata#Cbs.OData.WebAPI.Opleidingsniveau",
  "value": [
    {
      "Key": "2018700",
      "Title": "1 Basisonderwijs, vmbo, mbo1",
      "Description": "Het behaalde onderwijsniveau omvat het basisonderwijs, het vmbo, de eerste 3 leerjaren van havo/vwo, de entreeopleiding (mbo1) en het praktijkonderwijs.",
      "CategoryGroupID": 1
    },
    {
      "Key": "2018740",
      "Title": "2 Havo, vwo, mbo2-4",
      "Description": "Het behaalde onderwijsniveau omvat de bovenbouw van havo/vwo, de basisberoepsopleiding (mbo2), de vakopleiding (mbo3) en de middenkader- en specialistenopleidingen (mbo4).",
      "CategoryGroupID": 1
    },
    {
      "Key": "2018790",
      "Title": "3 Hbo, wo",
      "Description": "Het behaalde onderwijsniveau omvat de hbo- en wo-opleidingen (inclusief die leidend tot de doctorsgraad).",
      "CategoryGroupID": 1
    }
  ]
}
```

More generally, if you want to see how a dimension is built, you can take the base url of the dataset and add the dimension name as last segment. This is true for any dimension, including GeoDimension, GeoDetail and TimeDimension. Those are often found with endopoints like /RegioS, /WijkenEnBuurten, /Perioden etc.

### 6 - Understand region and time codes

Spatial and temporal dimensions (region and time) are standardized in CBS data as GeoDimension, GeoDetail and TimeDimension. There is a [dutch manual](https://www.cbs.nl/-/media/open-data/cbs-open-data-services.pdf) that documents the way geographical and time units are coded. On page 15 you find time codes, more on region codes can be found on page 20. 

#### Time codes

The codes for the temporal dimension can be understood by the manual, but if you want to know which TimeDimensions are used in a dataset, you can check either the /Perioden or /Periods endpoint like this:

```
https://opendata.cbs.nl/ODataApi/OData/83642NED/Perioden
```

To check what endpoint to use, check `/DataProperties`, here you should find it as one of the 'Key' values. It is also findable in `/$metadata`, in the `<EntityContainer>` section you find all valid endpoints. Or you can just visit the base url, here it is to be found as well.

#### Region codes

The region codes only make sense if you know which region code refers to which region. Using postfixes like `/Regions`, `/RegioS` and `/WijkenEnBuurten` provides listings of geographical units. 

Be aware that a table base url only supports this parameter if the table contains spatial data. Check the base url to see which postfixes are possible. Here are some useful links:

```
https://opendata.cbs.nl/ODataApi/OData/83642ENG/Regions  // all municipalities

https://opendata.cbs.nl/ODataApi/OData/86052NED/WijkenEnBuurten  // all districts and neighbourhoods

https://opendata.cbs.nl/ODataApi/OData/82211NED/RegioS  // provinces, 'landsdelen', 'corop-gebieden' and municipalities

https://opendata.cbs.nl/ODataApi/OData/83502NED/Postcode   // all postal codes
```

### 7 - Querying the (Un)TypedDataSet 

TypedDataSet and UntypedDataSet are use as last path segment and provide access to the actual data in the table. The difference can be found on page 23 and 24 of the [manual](https://www.cbs.nl/-/media/open-data/cbs-open-data-services.pdf), it comes down to the use of table info in calculations and visual representations, for which TypedDataSets is better suited. They provide the same content.

When requesting the full dataset by creating a base url plus /TypedDataSet the CBS server will most often protest, as you cannot retrieve more than 10K rows at once. Therefore you need to restrict the table output. Basically there are three main request methods that will help you customize the response. The dollar sign is mandatory:

- $top (eventually combined with $skip) 
- $select
- $filter

The specific Odata query language you need to use is a sort of derivative of sql but has its own syntax. It is described [here](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html). The CBS manual suggests that the official syntax allows more than what is implemented by CBS itself, but I cannot find more precise documentation on it. 

What I find useful, with regards to the application I am building, is the possibility to filter on substrings (using startswith or substring) so that it is possible to find only province, municipalities, wijken or buurten. It also allows for limiting the timeperiods for which you want data. Furthermore $select is convenient if you are only interested in specific fields. In the case that the number of rows you need is more than the allowed 10k, using $top and $skip helps to subdivide the resultset in smaller batches.

To illustrate what is possible, I created some working queries:

```
https://opendata.cbs.nl/ODataApi/OData/37230NED/TypedDataSet?$select=*&$filter=substring(RegioS,0,5) eq 'GM051'

https://opendata.cbs.nl/ODataApi/OData/37230NED/TypedDataSet?$filter=RegioS eq 'GM0518' and Perioden eq '2002MM02'

https://opendata.cbs.nl/ODataApi/OData/37230NED/TypedDataSet?$filter=RegioS eq 'GM0518' and substring(Perioden,0,4) eq '2002'

https://opendata.cbs.nl/ODataApi/OData/37230NED/TypedDataSet?$select=*&$top=10&$skip=20

https://opendata.cbs.nl/ODataApi/OData/37230NED/TypedDataSet?$filter=startswith(RegioS, 'GM03')&$top=3
```

The whitespaces and quotation marks in the url's are, I suppose, no problem, but if so eventually you can use some Java URI method to create a proper url. 











