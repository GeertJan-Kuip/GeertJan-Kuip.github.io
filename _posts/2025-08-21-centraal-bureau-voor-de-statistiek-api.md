# Centraal Bureau voor de Statistiek api

Currently I'm working on a few small projects for a portfolio that shows my knowledge and skill in Java and Spring. The first mini-project I'm working on is a one-page website with a single search field. You can type the name of a dutch town (it has an autofill similar to buienradar.nl) and get some statistics on demographics. Initially I wanted to add some data on available shops, but accessing the internal api's of retailers (Kruidvat, Etos, Trekpleister) via a Java Spring application was more of a pain than I expected. Those endpoints are protected and can only easily be accessed via a browser or apps like Postman and Boomerang.

## Pdok and CBS

[PDOK](https://www.pdok.nl/) is a platform providing dutch geodata. I used their api to retrieve a list of dutch towns and it worked fine. A link like [this](https://api.pdok.nl/bzk/locatieserver/search/v3_1/suggest?q=type:woonplaats) gives you the complete list, [this one](https://api.pdok.nl/bzk/locatieserver/search/v3_1/suggest?q=type:gemeente) provides all municipalities and if you want details of towns, you can do it with a link like [this](https://api.pdok.nl/bzk/locatieserver/search/v3_1/lookup?id=wpl-421d72725f03866e9aee43e333794774).

CBS (Centraal Bureau voor de Statistiek) is a different story. They have an [api](https://opendata.cbs.nl/ODataApi/) where you can do extensive search in a ton of tables, but probably due to the sheer amount of data, the large number of topics and dimensions, and the long period over which the data was collected, it is way harder to create url's that will return any data at all, let alone the data you were looking for.

## Decrypting the CBS Open Data portal

To get the information you want, you can take the following steps:

### 1 - Find the right table

Go to [website](https://opendata.cbs.nl/statline/portal.html?_la=nl&_catalog=CBS). You can also go [here](https://opendata.cbs.nl/statline/#/CBS/nl/navigatieScherm/thema), this one has better navigation. [This link](https://opendata.cbs.nl/ODataApi/) provides a full listing as well, without further descriptions.

### 2 - Check the table preview 

The preview that you will see shows just one of the many ways in which data can be ordered and subselections can be made. You can drag the variables around (I think there is a maximum of seven) and you can click the funnel symbols to make subselections within a variable (or call it a topic or a dimension). What you learn here is whether the table is able to generate the info you need. For example, if you want a fine grained geographical level (municipality or 'wijk'), you should be able to select municipalities or 'wijken' when you click a variable related to geography.

### 3 - Create the base url for the table api

The url of the table page contains a code, five digits followed by either 'NED' or 'ENG. Create a new url like this (the last parameter points to a specific table):

```
https://opendata.cbs.nl/ODataApi/OData/83642NED/
```

### 4 - Figure out variable names (1)

This url provides all the possible basic links to the data and metadata of the table. This is what it looks for table 83642NED:

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

Any of these links is valid and tells you more. The `/UntypedDataSet` and `/TypedDataSet` are the path segments that you will need for inquiry into the data, the others are for metadata. `/UntypedDataSet` and `/TypedDataSet` often return too much data rows and thus require refinement (more on that later). 

A useful link, sort of hack, is this one:

```
https://opendata.cbs.nl/ODataApi/OData/83642NED/TypedDataSet?$top=1
```

It provides a sort of definitive list of the variable names you can use in your queries. 

### 4 - Understanding Dimension and Topic

If you visit the `/DataProperties` endpoint you find a list of values that can be of the following odata types:

- Dimension
- Topic
- TopicGroup
- GeoDimension
- GeoDetail
- TimeDimension

The important thing here is to understand the difference between Dimension and Topic. Note that GeoDimension, GeoDetail and TimeDimension are all specialized and standardized dimension types, and that TopicGroup is a sort of dummy object that groups similar Topics.

The difference between Dimension and Topic is that the latter refers to the things that are measured, resulting in the numbers, codes or maybe boolean values that you find in the tables. A topic can be 'number of bankruptcies' or 'average income'. Dimension is the thing that specifies which items should be counted and which should not be counted for a given topic. For example, if you want to limit the measurement of average income to a certain age group, then average income is the topic and age group is the dimension. This explains why GeoDetail, GeoDimension and TimeDimension are dimensions and not topics, as we use them to restrict the number of things to count or measure to a specific period or geographical area.


### 5 - Region and time codes

Spatial and temporal dimensions (region and time) are standardized in CBS data as GeoDimension, GeoDetail and TimeDimension. There is a [dutch manual](https://www.cbs.nl/-/media/open-data/cbs-open-data-services.pdf) that documents the way geographical and time units are coded. On page 15 you find time codes, more on region codes can be found on page 20. 

While the codes for the temporal dimension can be understood by the manual, the region codes only make sense if you know which region code refers to which region. Using postfixes like `/Regions`, `/RegioS` and `/WijkenEnBuurten` provides listings of geographical units. Be aware that a table base url only supports this parameter if the table contains spatial data. Check the base url to see which postfixes are possible. Here are some useful links:

```
https://opendata.cbs.nl/ODataApi/OData/83642ENG/Regions  // all municipalities

https://opendata.cbs.nl/ODataApi/OData/86052NED/WijkenEnBuurten  // all districts and neighbourhoods

https://opendata.cbs.nl/ODataApi/OData/82211NED/RegioS  // provinces, 'landsdelen', 'corop-gebieden' and municipalities

https://opendata.cbs.nl/ODataApi/OData/83502NED/Postcode   // all postal codes
```




