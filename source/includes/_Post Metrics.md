# Post Metrics

> End Point prefix is **/api/v1/metrics**

Send data samples to Anodot using Metric 2.0/3.0 protocols. 
Both the version 2.0 and 3.0 protocols work with a **schema** - this means that the data is structured into key-value pairs and enables Anodot to better 'understand' and interact the metrics sent to it. 
The main difference between 2.0 and 3.0 is that 3.0 enable sending a '**watermark**' notification to Anodot, enabling a better detection and assignment of data points to a time frame.

<aside class="warning">
Using the Metrics 1.0 Protocol:</br>
In cases where the metrics cannot be represented in Metric 2.0 Format,
Anodot continues to support legacy formats such as Graphite and Graphite collectors, see Anodot Protocol 1.0 below. 
</aside>


**High Level Flow**

1. [Creating a Schema](#schema)
2. [Sending samples matching the schema](#send-data-samples)
3. [Sending watermarks timestamp to direct Anodot to process data samples up to the timestamp](#send-stream-watermark)


## Send Data Samples (3.0)

> Request Example - Sending metrics (3.0)

```shell
curl --location --request POST 'http://app.anodot.com/api/v1/metrics?protocol=anodot30&token={{data-token}}' \
--header 'Content-Type: application/json' \
--data-raw '[
 {
   "schemaId": "111111-22222-3333-4444",
   "timestamp": “143876178”,
   "dimensions": {
     "Geo": "US",
     "Device": "Mobile",
     "ProductCategory": "Shoes"
   },
   "measurements": {
     "measure1": “10”,
     "measure2": “25.5”
   },
   "tags": {
     "ActiveCampaignID": [
       “1234”
     ],
     "AccountManagers": [
       "JohnDoe",
       "MaryJane"
     ]
   }
 }
]'
```

Use this API to send metrics to Anodot based on a schema you're defined (Protocol 3.0)
Please note that the maximal number of entries is 10K per request. 

### Request

Field | Description
------|------------
schemaId | Id recieved from the create schema call
timestamp | Integer - The data sample's timestamp (Unix epoc time in seconds). This has to be at most one hour in the future. 
dimensions | an array of key-value pairs of the metric dimensions.
measurements | an array of a measurements - decimal double precision number, without a thousands seperator. 
tags | (Optional) List of tags attached to the measure. Key value pairs. Notice that tags are metadata of the metric and do not affect its uniqueness.

## Send Data Samples (2.0)

> Request Example - Sending metrics (2.0)

```shell
curl --location --request POST 'http://app.anodot.com/api/v1/metrics?protocol=anodot20&token={{data-token}}' \
--header 'Content-Type: application/json' \
--data-raw '[
  {
    "properties": {
      "what": "NumberPurchases",
      "Geo": "US",
      "Device": "Mobile",
      "ProductCategory": "Shoes",
      "target_type": "counter"
    },
    "tags": {
      "ActiveCampaignID": [
        1234
      ],
      "AccountManagers": [
        "JohnDoe",
        "MaryJane"
      ]
    },
    "timestamp": 143876178,
    "value": 58
  }
]'
```

Use this API to send metrics to Anodot based on a schema you've defined (Protocol 2.0)

### Request

Field | Description
------|------------
properties | list of properties of the metrics in a key-pair value array. This array should always contain a 'what' property. See notes below on best practices using this. 
tags | (Optional) List of tags attached to the measure. Key value pairs. Notice that tags are metadata of the metric and do not affect its uniqueness.
timestamp | Integer - The data sample's timestamp (Unix epoc time in seconds). This has to be at most one hour in the future. 
value | decimal double precision number, without a thousands seperator. 

Some notes on the 'properties' array sent using the protocol:

* Key value pairs should contain ascii characters only.
* The following characters are not allowed: “.”, “=” and space. (Remove them or replace then with "_".
* The "what" property must be set ("what" represents what is actually being measured).
* A metric may contain up to 20 properties.
* property key must be a non empty string no longer than 50 characters.
* property value must be a non empty string no longer than 150 characters.
* The combination of all property names and values determines the uniqueness of the metric in Anodot.
* The target_type property represents how samples of the same metric are aggregated in Anodot valid values are:</br>
gauge (average aggregation), counter (sum aggregation). (default is gauge)

## Send Data Samples (1.0)

> Request Example - Sending metrics (1.0)

```shell
curl --location --request POST 'http://app.anodot.com/api/v1/metrics&token={{data-token}}' \
--header 'Content-Type: application/json' \
--data-raw '[
  {
    "name": "company=anodot.device=test.what=hello_world_count",
    "timestamp": 1520325400,
    "value": 100,
    "tags": {
      "target_type": "counter"
    }
  }
]'
```

<aside class="warning">
Using the Metrics 1.0 Protocol:</br>
Please keep in mind that this protocol is deprecated. While it is still supported - we do not encourage its use.
</aside>

Use this API to send metrics to Anodot based on the 1.0 Metric protocol. You may use this protocol to create new metrics or submit data points for new or existing metrics. You can submit data points for multiple metrics in a single request. 

* As a rule of thumb it is recommended not to send more than 1000 samples in a single http request, although it can handle more but the latency will be higher.
* Data points should be sent in chronological order, otherwise data points which are out of order will be dropped.
* To assign one or more tags to a metric add the tags to the metric name with a prefix of a hashtag (#).

### Request

In the request body you need to send an array of time series data points. The data points have the following parameters: 

Field | Description
------|------------
name | (Mandatory) The unique identifying name of the metric being tracked. See below notes on metric names. 
timestamp | Integer - The data sample's timestamp (Unix epoc time in seconds). 
value | decimal double precision number, without a thousands seperator. 
tags | (Optional) List of tags attached to the measure. Key value pairs. Notice that tags are metadata of the metric and do not affect its uniqueness.

Legal Metric Names: 

* Any ascii printable character from the range 32-126
* except for ' (single quote} and space.
* The following characters will require using escape characters when searching for metrics in Anodot: + - ! ^ && [ ] { } < > ~ || " ? =
* Special characters: . - Separator between key/values pairs | # - metric tag – add a tag to a metric, doesn’t impact metric uniqueness | = - key value separator

## Send Stream Watermark

> Request Example - Sending a watermark request

```shell
curl --location --request POST 'http://app.anodot.com/api/v1/metrics/watermark?protocol=anodot30&token=8e4d630794fa670a874b2b3048b1f213' \
--header 'Content-Type: application/json' \
--data-raw '{
  "schemaId": "111111-22222-3333-4444",
  "watermark": “143877000”
}'
```

Use this API to send the watermark timestamp to Anodot (Only relevant for protocol 3.0). A watermark timestamp commits that no data samples with timestamps less than or equal to it will be sent.

### Parameters

Field | Description
------|------------
schemaId | Id recieved from the 'create schema' call
watermark (Integer) | timestamp of the watermark (Unix epoch time in seconds)

<aside class="success">
A Pro Tip:</br>
To close the bucket and send the data for processing you need to send a beginning of the next bucket as a watermark timestamp. For example, you have hourly data and you’ve just sent data points with `2020-06-01 06:00` timestamp. To commit it to processing you need to send `2020-06-01 07:00` watermark (the beginning of the next hourly bucket). Note that flushing the data is a scheduled process and you will probably need to wait some time (around 15 minutes) until you see a data point in the UI after sending the watermark.
</aside>
