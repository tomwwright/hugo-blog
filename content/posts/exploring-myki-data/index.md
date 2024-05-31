---
title: 'Exploring Melbourne Myki Data with AWS Athena'
description: Simple analysis of CSV data in S3 with serverless SQL queries
date: 2018-09-28T16:32:41+10:00
tags: ["AWS", "Data", "Tech"]
---

As a Melbourne resident and daily commuter on our Myki public transport fare system (no comment), I was intrigued when I heard the dataset for the[ Melbourne Datathon 2018](http://www.datasciencemelbourne.com/datathon/) was to be large scale, real world Myki usage data. What cool insights can we glean on how our bustling city uses its public transport network? Let’s find out! Best of all, we’ll check it out without transforming it from CSV, or even moving it out of S3.

<!--more-->

> **NOTE**
> 
> This article is a reposting of my 2018 [Medium](https://medium.com/towards-data-science/exploring-melbournes-myki-data-with-aws-athena-511410d2e461) article. 
{ .note }

> **INFO**
> 
> At the request of Data Victoria, links to the data discussed in this article have been removed.
{ .info }

Here are a couple of quick stats I gleaned from this 1.8 billion row dataset, with SQL queries that run in seconds, for much less than the cost of a cup of coffee:

- Fares consisted of `65.65%` full-fare, `33.65%` paying concession fares, and a measly `0.69%` free-riding concessions
- The favourite touch-off location for Victorian Seniors Concession card holders on a Sunday is Box Hill Railway Station

## AWS Athena

[AWS Athena](https://aws.amazon.com/athena/) is a fully-managed, serverless query service that allows you to run SQL queries against data stored in S3 buckets — it’s a bit like magic. Built on [Apache Hive](https://hive.apache.org/) and Facebook’s [Presto](https://prestodb.io/), Athena let’s you define (or even discover) the schema of your data, then start running queries instantly. And all you pay for is the data that Athena has to riffle though to run your query, and the cost of storing it in S3.

So first let’s get our data, pop it into S3, and prepare our schema.

## Preparing our Myki data in S3

To have a first look and explore the data files, check out this public S3 link:

```
Edit: by request of Data Victoria, links to the data discussed in this article have been removed.
(bucket contents is 25.4gb)

---

Once again, many thanks to Data Science Melbourne for organising the Melbourne Datathon and providing the data. Thanks also to PTV for releasing the dataset.

http://www.datasciencemelbourne.com/datathon/

https://www.ptv.vic.gov.au/
```

Here is a brief description of the files we’re interested in for today:

```bash
/
events/         # our myki usage dataset is in here
 Samp_0/        # data is split into ten chunks
  ScanOffTransaction/  # myki 'touch on' events when people board
  ScanOnTransaction/   # myki 'touch off' events from disembarking
   2015/        # year partitions
   ...
   2017/
    Week1/      # week partitions
    ...
    Week52/
     QID3530164_20180713_12510_0.txt.gz    # data!
 Samp_1/
 ...
 Samp_9/
calendar/       # map dates to week numbers, days of week, etc.
cardtypes/      # myki fare types: full fare, concession, etc.
stops/          # links stop information to a stop id
```

Create yourself a bucket and copy in the data! _(feel free to just copy one of the `Samp_X` directories if you want to have a leaner look at the data)_

```bash
aws s3 mb s3://mediumreader-myki-data
aws s3 cp --recursive s3://bucket-no-longer-exists/ s3://mediumreader-myki-data/
```

Easy! Now, we tell AWS Athena what our data looks like so it can query it — we do this using our Data Catalog in AWS Glue, which integrates with Athena.

## Preparing our data schema in AWS Glue Data Catalogue

[AWS Glue](https://aws.amazon.com/glue/) is Amazon's fully-managed ETL (extract, transform, load) service to make it easy to prepare and load data from various data sources for analytics and batch processing. Today we’re just interested in using Glue for the Data Catalogue, as that will allow us to define a schema on the Myki data we just dumped into S3. AWS Glue makes this easy by providing data “crawlers” that can peek at our data and discover a lot of our schema automatically.

- Navigate to `AWS Glue`
- Select `Crawlers` from the menu
- Select `Add crawler`

{{<figure src="./aws-glue-crawler.png" title="Getting started by adding a crawler to discover our schema" >}}

- Name our crawler `myki_crawler`
- Click `Next`, we don’t need any of the optional configuration here

{{<figure src="./aws-glue-crawler-2.png" title="Naming our crawler, other optional config is fine as defaults" >}}

- Configure `S3` data store
- `Specified path in my account`
- Include path for me is `s3://tomwwright-myki-data/`, replace with whatever you called your bucket earlier when you copied the data
- Leave `Exclude patterns` blank, for this dataset we’ll just clean up any junk the crawler finds that we don’t want
- Click `Next`, select `No` to adding another data store, click `Next` again

{{<figure src="./aws-glue-crawler-3.png" title="Pointing our crawler at our S3 data bucket" >}}

- Select `Create an IAM role` (unless you have roles in place for Glue)
- Enter `MykiCrawler` for the name — _or whatever you want, I’m a Medium post, not your mother_
- Click `Next`

{{<figure src="./aws-glue-crawler-4.png" title="Creating an IAM role to give our crawler access to our bucket" >}}

- Select `Run on Demand` as the frequency, we just want to fire this off once
- Click `Next`

{{<figure src="./aws-glue-crawler-5.png" title="We just want to run it the once anyway…" >}}

- Select `Add database`, let’s just call it `myki`, don’t need any of the optional config, click `Create`
- Leave all the optional config as is
- Click `Next`
- Click `Finish`!

{{<figure src="./aws-glue-crawler-6.png" title="Creating a database for the tables discovered by the crawler" >}}

You should have been returned to the Crawlers screen of AWS Glue, so select `myki_crawler` and hit `Run crawler`. It’ll take about 7 minutes to run, in my experience, so maybe grab yourself a coffee or take a quick walk.

Once the crawler has completed we’ll be able to view the discovered tables under `Databases` -> `Tables`. The first thing we’ll do is delete some junk tables that the crawler has created from examining some of the non-data files in our S3 bucket — delete all the tables with the classification `Unknown`, as in the screenshot below. We could have saved ourselves this hassle by neglecting to copy them into the bucket in the first place, or added them to the crawler’s exclude paths configuration. Oh well, easy enough to just delete them.

{{<figure src="./deleting-junk-tables.png" title="Deleting junk tables the crawler inadvertently created by scanning everything in the bucket — that’s our bad, oops." >}}

Opening up the calendar table, we can see all the great stuff the crawler has discovered for us: data format, delimiters, record counts, column types, and so on. One thing that is missing are the column names, because that information isn’t present in the myki data files. Below you’ll find some column labels (not necessarily all of them) that we need to apply in order to be able to write readable queries for our tables. Select a table and click `Edit schema` in the top right to update the columns.

```
table: calendar

1  col0   bigint   dateid
2  col1   string   date
3  col2   bigint   year
6  col5   string   month
13 col12  string   daytype  # 'Weekday', 'Weekend', 'Public Holiday'
16 col15  string   dayofweek

---

table: cardtypes

1  col0   bigint   typeid
2  col1   string   typedesc
3  col2   string   paidorfree  # 'Paid', 'Free'
4  col3   string   concession  # 'Full Fare', 'Concession'

---

table: events

1  col0   bigint   mode
2  col1   string   date
3  col2   string   datetime
4  col3   bigint   cardid
5  col4   bigint   cardtypeid
6  col5   bigint   vehicleid
8  col7   string   routeid
9  col8   string   stopid
10 partition_0     samplenum
11 partition_1     onoroff  # 'ScanOffTransaction', 'ScanOnTra...on'
12 partition_2     year
13 partition_3     week

---

table: stops

1  col0   bigint   stopid
2  col1   string   name
3  col2   string   longname
4  col3   string   stoptype
5  col4   string   suburb
6  col5   bigint   postcode
10 col9   double   lat
11 col10  double   long
```

With our column names in place, we’re finally ready to do some querying!

## Querying our schema with AWS Athena

Clicking through to the Athena service we find ourselves first on the Query Editor screen. Cool, let’s select our `myki` database and plug in a simple count query!

{{<figure src="./query-card-types.png" title="AWS Athena’s Query Editor makes it simple to develop queries against our Myki schema" >}}

When we’ve got a query we’re really happy with we can hit `Save as` and give it a name, then we can easily find it and run it again under `Saved Queries`. Athena also allows us to access our query history under `History`, where we can download any query results.

{{<figure src="./query-history.png" title="AWS Athena’s query history allows us to find our past queries and download the results" >}}

### Partitions

An important metric to notice here for your Athena queries is the **data scanned**, as [pricing for Athena](https://aws.amazon.com/athena/pricing/) is directly (and only) related to the amount of data read by Athena in the process of running your query:

> “You are charged for the number of bytes scanned by Amazon Athena, rounded up to the nearest megabyte, with a 10MB minimum per query.
> ...
> $5 per TB of data scanned”

As it operates on plain flat files it is difficult for Athena to be intelligent about the data that it scans, and generally it will need to read all the files that your schema specifies as making up the table. Unless, when creating the schema, **partitions** are specified that define “columns” that map to directories that Athena can use to be intelligent about what data should be scanned for a given query. To illustrate, recall the folder structure of our data:

```bash
/
events/         # our myki usage dataset is in here
 Samp_0/        # data is split into ten chunks
  ScanOffTransaction/  # myki 'touch on' events when people board
  ScanOnTransaction/   # myki 'touch off' events from disembarking
   2015/        # year partitions
   ...
   2017/
    Week1/      # week partitions
    ...
    Week52/
     QID3530164_20180713_12510_0.txt.gz    # data!
 Samp_1/
 ...
 Samp_9/
calendar/       # map dates to week numbers, days of week, etc.
cardtypes/      # myki fare types: full fare, concession, etc.
stops/          # links stop information to a stop id
```

Our events folder is structured by a series of subfolders that neatly divide our events data by a couple of key columns: a numbered sample folder, whether it is a touch-on or touch-off event, the year, and the week. Luckily, our AWS Glue crawler was smart enough to notice all that, and has already organised all that in our schema! Nice!

{{<figure src="./partitions.png" title="You can view partitions for a table in the AWS Glue Data Catalogue" >}}

To illustrate the importance of these partitions, I’ve counted the number of unique Myki cards used in the year 2016 (about 7.4 million, by the way) with _two different queries_:

- one using a `LIKE` operator on the date column in our data,
- and one using our `year` partitioning column.

{{<figure src="./partitions-2.png" title="The power of partitioning: Athena scans only the relevant data for 2016, less than 30% of the entire dataset" >}}

Our first query, using date `LIKE '2016%'`, has to scan the entire events table, all 25 gigabytes of it, because Athena is unable to determine what files might have matching lines in them. Cost for this query is about **$0.12**.

Our second query, using `year = '2016'`, is able to retrieve our results and only scan 8 gigabytes of our events data! This is because `year` is one of the partition columns that maps to particular key patterns in our S3 data. Cost for this query is about **$0.04**, and runs in under one-third the time!

Intelligently organising your data with parititions is key to being able to run quick and cost-effective queries on Athena. Alright, let’s move on!

### Popular stops for touch-off

So, my first thought, where do people _go_ using their Myki? Seems like an easy enough question to answer with SQL, let’s start by getting a list of the most popular stops.

```sql
-- top 20 stop ids for touch-off events 
select stopid, count(*) as count
from events
where onoroff = 'ScanOffTransaction'
group by stopid
order by count desc
limit 20;
```

{{<figure src="./query-touch-offs.png" title="Touch-off events aggregated by the stop ID" >}}

Cool, we can join this with our `stops` table for a more human-readable representation, let’s do that.

```sql
-- select our top stops like before, then join to our stops table
with topstops (stopid, count) as (
  select stopid, count(*) as count
  from events
  where onoroff = 'ScanOffTransaction'
  group by stopid
)
select topstops.stopid, stops.longname, stops.suburb, topstops.count
from topstops left outer join stops
on topstops.stopid = stops.stopid
order by topstops.count desc
limit 20;
```

{{<figure src="./query-touch-offs-2.png" title="Results of running our “top touch-off stops” query" >}}

Hmm. There is a bunch of `stopid` rows that are missing `stops` information… To me, given they’re most of the most popular stops and there is the right number, my guess is the `64xxx` stops are the City Loop railway stops, and maybe `24001` is _Richmond station_?

Apart from that, these results make sense to me, not surprising at all to see all the top stops be railway stops.

{{<figure src="./city-loop.png" title="Melbourne’s City Loop — map courtesy of [Public Transport Victoria](https://static.ptv.vic.gov.au/PDFs/Maps/Network-maps/1535090671/PTV_MetropolitanTrainNetworkMap_August2018.pdf)" >}}

### Breakdown of fare types

There always seems to be a bit of chatter about the spending on providing concession public transport travel to various groups. What is the breakdown of full-fare vs. concession? Again, seems simple enough. Let’s start by aggregating on the card type as present in our `events` table.

```sql
-- aggregate our touch-on events by the card type used
select cardtypeid, count(*) as count
from events
where onoroff = 'ScanOnTransaction'
group by cardtypeid
order by count desc
```

{{<figure src="./query-fare-types.png" title="Touch-on events aggregated by the card type ID" >}}

Nice, so same as last time, let’s join to our `cardtypes` table to deliver some more human-readable information:

```sql
-- aggregate by card type as before, then join to card types
-- information to determine concession and paying status
with cardtypecounts (cardtypeid, count) as (
  select cardtypeid, count(*) as count
  from events
  where onoroff = 'ScanOnTransaction'
  group by cardtypeid
)
select cardtypecounts.cardtypeid, cardtypes.typedesc, cardtypes.concession, cardtypes.paidorfree, cardtypecounts.count
from cardtypecounts left outer join cardtypes
on cardtypecounts.cardtypeid = cardtypes.typeid
order by cardtypecounts.count desc;
```

{{<figure src="./query-fare-types-2.png" title="Top fare types and concession / paying status" >}}

Full-fare passengers look like an overwhelming majority, at more than 4 times the most common (but vaguely-labelled) `General Concession` concession fare type. But then the rest of the list is made up of the various distinct concession types, so it might not be so simple. The only free-riding fare type to crack this Top 14 is the `Employee Travel Pass` — makes sense.

Let’s go for an even higher-level look by aggregating again, I’m curious about the overall break-down of full-fare vs. concession vs free.

```sql
-- further aggregating our query by concession and paying
-- statuses
with cardtypecounts (cardtypeid, count) as (
  select cardtypeid, count(*) as count
  from events
  where onoroff = 'ScanOnTransaction'
  group by cardtypeid
)
select cardtypes.concession, cardtypes.paidorfree, sum(cardtypecounts.count) as count
from cardtypecounts left outer join cardtypes
on cardtypecounts.cardtypeid = cardtypes.typeid
group by cardtypes.concession, cardtypes.paidorfree 
order by count desc;
```

{{<figure src="./query-fare-types-3.png" title="Full-fare vs. concession vs. free fare types" >}}

Alright, there is the full picture: 65.65% full-fare, 33.65% paying concession fares, and a measly 0.69% free-riding concessions.

### Top Sunday destinations for Seniors

Finally, let’s try something a little specific. On a Sunday, where do our senior public transport users like to go?

Joining to our `calendar` table will allow us to determine which dates are `Sunday`, and our query on fare types before shows us that `cardtypeid = 9` is the `Victorian Seniors Concession` fare type we need to be looking for. Awesome!

```sql
-- selecting touch-on events that were on a Sunday using a Victorian Seniors
-- Concession fare
select stopid, count(*) as count
from events, calendar
where onoroff = 'ScanOffTransaction'
and events.date = calendar.date
and events.cardtypeid = 9
and calendar.dayofweek = 'Sunday'
group by events.stopid;
```

{{<figure src="./query-senior-sundays.png" title="Touch-on events on a Sunday by seniors, aggregated by the stop ID" >}}

Now just the same as our first query, join to stops to give us some human-readable information. Remembering again that some `stops` seem to be missing their information…

```sql
-- join our sunday seniors stops to stops information
with seniorsundaystops (stopid, count) as (
  select stopid, count(*) as count
  from events, calendar
  where onoroff = 'ScanOffTransaction'
  and events.date = calendar.date
  and events.cardtypeid = 9
  and calendar.dayofweek = 'Sunday'
  group by events.stopid
)
select seniorsundaystops.stopid, stops.longname, stops.suburb, seniorsundaystops.count
from seniorsundaystops left outer join stops
on seniorsundaystops.stopid = stops.stopid
order by seniorsundaystops.count desc;
```

{{<figure src="./query-senior-sundays-2.png">}}


Our mysterious unnamed stops (almost certain it is the City Loop stops) remain head of the pack, but apparently our Myki-wielding Seniors are fond of spending time in _Box Hill_, _Caulfield_, and _Footscray_. Who knew.

## Wrapping up

It’s good to take a moment and think about how cool this sort of technology is, because it is easy to forget once you dive in. The ability to be running SQL queries, with joins, against nothing more than CSV files sitting in S3. That’s impressive.

This Myki data is also not a trivial amount to be analysing. Over 1.8 billion rows, and we’re able to crunch it in less than 10 seconds. And it costs absolute pennies. The queries listed in this article all-up scans about 145 gigabytes of S3 data, costing me a whopping total of…

$0.75

Happy querying!

--Tom

{{<figure src="./query-events.png" title="Counting our data — it’s a decent bit to be crunching in under 10 seconds and paying next to nothing for" >}}



