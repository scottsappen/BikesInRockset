# Mountain Bike Stuff In Rockset
This was just an excuse for me to use the Rockset console.

If I were to wish someone who scanned this to walk away with anything, it might be these 3 points:
1. Simple - I ingested and queried data with minimal effort using a slick web UI.
2. Speed - I used a small dataset, sure, but innovative indexing combined with separation of storage and compute means fast.
3. Scaling - I was freed of operational woes. Zero to hero in my own mind fast.

Althought I'd understand if you said, wow, could you have used any less of an interesting data set?

I'm so used to having to login to EC2 to do something with my VMs or using a UI that breaks or not being able to see all my operational metrics and billing in one spot. I didn't have any of those issues with Rockset. Of course, you will want to integrate external Prometheus+Grafana or similar monitoring capabilities in real workloads, for instance, but out of the box I had a web UI to view latency, ingestion throughput, data storage consumption and cost. Pretty cool.

<br/>

# The Source Data
I could have used one of Rockset's [Quick Start](https://docs.rockset.com/documentation/docs/quickstart) data sets, but I decided to try something else on a whim. I wondered if there was a way to download the data from [my hobby website](https://buildamountainbike.com/) and import it into Rockset.

I logged into my hobby site and found an area in the admin section on how to download the data. It saved as an ugly [XML file](https://github.com/scottsappen/BikesInRockset/blob/main/onemountainbike.wordpress.2024-01-11.000.xml). Ha! I haven't worked with XML in ages. As you can see from this in-line screenshot, it's pretty ugly. I paused for a moment unsure if I really wanted to mess around with this data, but decided to continue on anyway.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/XMLSource.jpg)

<br/>

# Rockset supports file uploads and XML natively
Any excuses I had on not working with XML went out the window with a quick Google search. Rockset natively supports a number of file types upon import. I was intrigued at the root and document [tag options](https://rockset.com/docs/file-formats/#xml-files) upon inmport.

<br/>

# Rockset Collection
First things first, creating a trial account with Rockset could not have been easier. It's point and click. Next, you can create a Collection of data in Rockset upon which you can query. Signing up is developer friendly for a while (free) and didn't even require a credit card. Everything works splendidly and in minutes I was ready to upload data.

I played around a tiny bit and found "item" in my source file as a good starting point so I typed it into the Document tag and let the ingest sample preview show me the goods.

If this is any indication, I can certainly imagine how easy it would have been to import this file had it been sitting in cloud storage like GCS or a S3 bucket.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/XMLdataimport.jpg)

<br/>

# Transform On Ingest
The data transformations on ingest make it super simple to edit and trim your data set before importing. I'm sure with UDFs and other techniques, you can import super clean data.

```EXCEPT``` makes it easy to discard data so you don't waste space (or indexes).
<br/>
```EXTRACT``` and ```TRY_CAST``` were convenient date/time parse and format functions since I did not need timestamps.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/TransformOnIngest.jpg)

<br/>

# Dynamic Scaling
I couldn't do a bulk ingest without scaling my VI to a medium size. Rockset took care of that on the fly for me. I didn't have to do anything, but "ok" it.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/EmailDynamicScalingConfirm.jpg)

<br/>

# Upload Complete
My data file started importing quickly into my new Collection.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/CollectionCreatedSuccessfully.jpg)

<br/>

# Watching Metrics
I started watching metrics and could tell as soon as my data was successfully imported.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/MetricsView1.jpg)
<br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/MetricsView2.jpg)
<br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/MetricsView3.jpg)
<br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/MetricsView4.jpg)

<br/>

# I Am Broke Now
Ha! I'm kidding. I mentioned developer friendly for a while (free).
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/UsageAndCost.jpg)

<br/>

# Dirty Data
Now I can start seeing what data is in the system.

In hindsight, I should have left out additional fields such as WPCATEGORY from the data ingest, but I missed it. I almost went back to clean it up since it's a cake walk, but decided to leave it and mess around with the dirty data on the output side with queries instead.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/DirtyData.jpg)

<br/>

# SQL Woes
My SQLServer, Oracle, Postgres, whatever days were coming back to haunt me trying to use functions that didn't exist like ```YEAR```.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/QueryNotSupported1.jpg)
<br/><br/>
It's all good though because the query editor tells you that you are not being smart and the documentation quickly leads me to other functions worth trying like ```EXTRACT (YEAR...```. There's probably a half dozen or more ways to skin this, but it's easy to try options in the editor.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/QueryNotSupported2.jpg)

<br/>

# Various Outputs
Now you can start playing around with a variety of queries. I started to explore new ones (to me) like UNNEST with some of that dirty data. Powerful array features in Rockset!
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/QueryResults.jpg)
<br/><br/>
Here I tried pulling out that nested array/object field starting off with a basic nested query using ```WITH```.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/QueryResults2.jpg)
<br/><br/>
And then I used ```UNNEST``` to peel out how many times those nicenames were used in descending order.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/QueryResults3.jpg)
<br/><br/>

<br/>

# Query Profiler
I nearly forgot I am going to want to spend time in here to see what's happening. I think Rockset is telling me I did something non-optimal. I'll have to investigate! :)
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/RocksetEfficiencies.jpg)

<br/>

# Query Lambdas
I also happened upon this in the UI and decided to give it a whirl. I was shocked how easy it was to create a public facing endpoint for my dataset and query! Check out how I turned that last SQL query into an endpoint.
<br/><br/>
First, I used the console to create a private API key that I will use to call the API endpoint. That's straightforward, it's under API Keys.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/RocksetAPIKey.jpg)
<br/><br/>
Next, I'll save my SQL query as a lambda and give it a name.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/RocksetQueryLambda.jpg)
<br/><br/>
Rockset makes it easy to integrate the API into an application or just testing it out. In this case, I'll use the provided CURL.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/RocksetQueryLambda2.jpg)
<br/><br/>
I decided to create a shell script to paste my API key into and run it from the command line.
```shell
scottsm1home@scotts-mbp Desktop % cat sapprocksetlambdacall.sh 
curl --request POST \
--url https://api.use1a1.rockset.com/v1/orgs/self/ws/SappenfieldWP_Bikes/lambdas/Bikes_NiceNamesInPosts_QL/tags/latest \
-H "Authorization: ApiKey YOUR_API_KEY_HERE" \
-H 'Content-Type: application/json' \
 | python3 -m json.tool
```

```shell
scottsm1home@scotts-mbp Desktop % ./sapprocksetlambdacall.sh
```

And voila, out comes the API call output. Easy peasy!

```shell
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1319  100  1319    0     0   1536      0 --:--:-- --:--:-- --:--:--  1535
{
    "collections": [
        "SappenfieldWP_Bikes.WPBikes_Collection"
    ],
    "aliases": [],
    "column_fields": [
        {
            "name": "countOfTags",
            "type": ""
        },
        {
            "name": "nicename",
            "type": ""
        }
    ],
    "results": [
        {
            "countOfTags": 19,
            "nicename": "sports"
        },
        {
            "countOfTags": 18,
            "nicename": "mtb"
        },
        {
            "countOfTags": 18,
            "nicename": "mountain-biking"
        },
        {
            "countOfTags": 18,
            "nicename": "outside"
        },
        {
            "countOfTags": 17,
            "nicename": "health"
        },
        {
            "countOfTags": 17,
            "nicename": "exercise"
        },
        {
            "countOfTags": 16,
            "nicename": "mental-health"
        },
        {
            "countOfTags": 16,
            "nicename": "nature"
        },
        {
            "countOfTags": 15,
            "nicename": "healthy-living"
        },
        {
            "countOfTags": 13,
            "nicename": "calories"
        },
        {
            "countOfTags": 2,
            "nicename": "injuries"
        },
        {
            "countOfTags": 2,
            "nicename": "crashes"
        },
        {
            "countOfTags": 2,
            "nicename": "accidents"
        },
        {
            "countOfTags": 1,
            "nicename": "apple"
        },
        {
            "countOfTags": 1,
            "nicename": "final-cut-pro"
        },
        {
            "countOfTags": 1,
            "nicename": "photoshop"
        },
        {
            "countOfTags": 1,
            "nicename": "flexibility"
        },
        {
            "countOfTags": 1,
            "nicename": "camera"
        },
        {
            "countOfTags": 1,
            "nicename": "photography"
        },
        {
            "countOfTags": 1,
            "nicename": "gopro"
        },
        {
            "countOfTags": 1,
            "nicename": "wind"
        },
        {
            "countOfTags": 1,
            "nicename": "graphics"
        },
        {
            "countOfTags": 1,
            "nicename": "fun"
        }
    ],
    "query_id": "9bd65de5-182d-428a-ac3a-0acbd63ec75b:w3mwCsQ:0",
    "query_lambda_path": "SappenfieldWP_Bikes.Bikes_NiceNamesInPosts_QL::f0e09cee14a44d8a",
    "stats": {
        "elapsed_time_ms": 22,
        "throttled_time_micros": 0
    },
    "status": "COMPLETED"
}
```
And like everything else in Rockset, it's easy to go see some metrics.
<br/><br/>
![](https://github.com/scottsappen/BikesInRockset/blob/main/imgs/RocksetQueryLambda3.jpg)

<br/>

# Some Final Thoughts
Well, I am just getting started and will continue to play around until I wear out my welcome.

Also, I guess I'll go back to my starting bullet points and see if I need to ammend them:

1. Simple - I ingested and queried data extremely with minimal steps and effort using a slick web UI.
2. Speed - I used a small dataset, sure, but innovative indexing combined with separation of storage and compute means fast.
3. Scaling - I was freed of operational woes. Zero to hero fast.

Nope.
<br/><br/>
