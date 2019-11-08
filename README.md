# Theia Search API Documentation

## Table of content

- [Authentication](#authentication)
- [Data Dictionaries](#data-dictionaries)
  - [Custom Dictionary](#custom-dictionary)
  - [TSV Dictionary Files](#tsv-dictionary-files)
  - [Update a Dictionary](#update-a-dictionary)
  - [Delete a Dictionary](#delete-a-dictionary)
- [Tagging](#tagging)
  - [Tagging content files](#tagging-content-files)
  - [Tagging Web URLs](#tagging-web-urls)
  - [Tagging Data Streams](#tagging-data-streams)
  - [Extracting Typed Values](#extracting-typed-values)
- [Search](#search)
- [Export](#export)


### Authentication

All Calls to protected API end points must include a header `X-Api-Key` with a valid API key.

#### Request

```
  -H 'X-Api-Key: API_KEY' 
```
If API key is valid than the response will be `HTTP 200` containing user profile data.

If API key is missing or not valid the endpoint will return `HTTP 401`:

#### Response

```
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```


### Data Dictionaries

Data dictionaries are used for labeling. **Theia** will search for dictionary values in input utterances and label the matching utterance with the dictionary key:

Dictionary (one item):
M&A => merger and acquisition

Input utterance:

>Merger and acquisition report published in 2019.

The above will be labeled with `M&A`

New dictionaries can be created in two ways: 
- By providing dictionary entries in the request payload
- By uploading a TSV file containing dictionary entries in the format of `key<TAB>value`.


#### Custom Dictionary

Create data dictionaries via `/dictionaries` end point as follows:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/dictionaries \
  -H 'X-Api-Key: SECRET_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
	"name": "My dictionary",
	"entries": 
	[
		{
			"key": "M&A",
			"value": "merger and acquisition"
		},
		{
			"key": "M&A",
			"value": "take over"
		}
	]
}'
```


#### Response

```
{
    "id": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
    "key": "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
    "name": "My dictionary",
    "global": false,
    "entries": [
        {
          "key": "M&A",
          "value": "merger and acquisition"
        },
        {
          "key": "M&A",
          "value": "take over"
        }
    ]
}
```



#### TSV Dictionary Files

Upload TSV data dictionaries via `/dictionaries/upload` endpoint as follows:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/dictionaries/upload \
  -H 'X-Api-Key: SECRET_API_KEY' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
  -F 'name=Uploaded dictionary' \
  -F file=@/home/files/custom_dictionary.tsv
```


To retrieve an available dictionary perform the request below, by providing the dictionary ID:

#### Request

```
curl -X GET \
 http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
 -H 'X-Api-Key: SECRET_API_KEY'
```


#### Update a Dictionary

To update an existing dictionary:

#### Request

```
curl -X PUT \
  http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'X-Api-Key: SECRET_API_KEY'
  -d '{
	"name": "Custom dictionary updated",
	"entries": 
	[
		{
			"key": "TBD",
			"value": "To be done"
		},
		{
			"key": "TBD",
			"value": "To be decided"
		}
	]
}'
```

#### Delete a Dictionary

To delete an existing dictionary:

#### Request

```
curl -X DELETE \
  http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'X-Api-Key: SECRET_API_KEY'
```


#### List Dictionaries


To list all existing dictionaries:

#### Request

```
curl -X GET \
  http://search.api.quantxt.com/dictionaries \
  -H 'X-Api-Key: SECRET_API_KEY'
``` 

#### Response

```
[
    {
        "id": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
            "key": "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
            "name": "My dictionary",
            "global": false,
        "entries": []
    }
]
```


### Tagging

Tagging (or labeling) is the process of identifying and labeling entities found in the unstructured content. Tagging can be done using unsupervised models as well as using user-defined data dictionaries. We will cover both approaches in below.


#### Tagging content files

First, upload all content files for tagging via the following call:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/file \
  -H 'X-Api-Key: SECRET_API_KEY' \
  -F file=@/Users/file.pdf
```

PDF, TXT and HTML formats are supported.

#### Response

```
{
    "uuid": "c351283c-330c-418b-8fb7-44cf3c7a09d5",
    "fileName": "file.pdf",
    "link": "http://portal.document.quantxt.amazonaws.com/user@example.com/c351283c-330c-418b-8fb7-44cf3c7a09d5",
    "date": "2019-10-25T20:14:41.925+02:00",
    "contentType": "application/pdf",
    "source": "file.pdf"
}
```

Then you can tag data via data dictionaries:

#### Request

```
curl -X POST \
  http://search.quantxt.com/search/new \
  -H 'X-Api-Key: SECRET_API_KEY' \
  -d '{
  "title": "My search with files and dictionaries",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "dictionaries": ["user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz"]
}'
```

`title` is optional but it is highly recommended for easier distinction between different tagging jobs.

There is no limit on the number of files and dictionaries that can be tagged via `/new` end-point.

#### Response

```
{
    "index": "puvqrjfhqq"
}
```

`index` represents the unique identification for the container that holds output labeled data. 


**Request parameters:**

`get_phrases`
(Optional, boolean) if `true` it will use built-in Theia Entity Tagging engine.

`excludeUttWithoutEntities`
(Optional, boolean) if `true` the output only includes utterances that have at least one tag from the input dictionaries.

`runSentenceDetect`
(Optional, boolean) if `true` it will break the input content unit into semantic sentences and run tagging at the sentence level.


To delete a data container:

#### Request

```
curl -X DELETE \
  http://search.api.quantxt.com/search/puvqrjfhqq \
  -H 'X-Api-Key: SECRET_API_KEY'
```


#### Tagging Web URLs:

Tagging can be performed on a list of URLs. All parameters in tagging files are applicable here.

#### Request

```
curl -X POST \
  http://search.quantxt.com/search/new \
  -H 'X-Api-Key: SECRET_API_KEY' \
  -d '{
   "title": "My search with URLs",
   "urls": ["https://electrek.co/2019/10/29/tesla-model-3-first-electric-car-approved-nyc-yellow-cab/", 
            "https://www.cnbc.com/2019/10/20/electric-car-prices-finally-in-reach-of-millennial-gen-z-buyers.html"]
}'
```


#### Tagging Data Streams

Tagging data from data streams such as third party APIs is supported. Please contact <support@quantxt.com> for details.



#### Extracting Typed Values

Data dictionaries allow the user to quickly search and label thousands of phrases in unstructured content. There are cases when users want to label a keyword or phrase as an entity only if it is associated with a value. For example:

A "release => "Manufactured" dictionary item will label both of the following utterances:

> The first automobile in the US **released** by Ford
> The first automobile in the US **released** in 1908 by Ford

However, if the user is looking for only release year of the car makers the first utterance won't have much use for him. He can only label the second utterance using a **Typed Dictionary**.

Supported dictionary types at the moment are:

`INT, REGEX, DOUBLE, DATETIME, NOUN, VERB, PERCENT, MONEY`

If a Type is set, a dictionary item will be labeled only if a type is found in its close proximity.
In the above example, user can set the Type to `INT` to identify **1908** that is in close proximity of **released**

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/new \
  -H 'X-Api-Key: SECRET_API_KEY' \
  -d '{
  "title": "My search with dictionaries and types",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "dictionaries": ["user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz||INT"]
}'
```


### Search


The Search endpoint allows user to run full-text and [faceted search](https://en.wikipedia.org/wiki/Faceted_search) in the labeled data.

#### Request

```
curl -X GET \
  http://search.api.quantxt.com/search/puvqrjfhqq \
  -H 'X-Api-Key: SECRET_API_KEY' \
```

**Request parameters:**

`q`
(Optional, string) Search query that goes againest the main content `title` field. It supports boolean `OR`, `AND` and `NOT` parameters.

`f`
(Optional, string) Query filters and must be used in pairs. Filters are created for each input dictionary. For example to include results that have one or more label from `Vehicle` dictionary the request should look like: `&f=Vehicle&f=*`. To include results that are labeled with `Ford` or `BMW` from the `Vehicle` dictionary, the request would be `&f=Vehicle&f=BMW&f=Vehicle&f=Ford`

`from`
(Optional, int) Offset for paging results. Dafaults to 0. Each page contain 20 items.


#### Response

```
{
    "Total": 2610,
    "results": [
        {
            "title": "The Federal Reserve Bank of New York provides gold custody to several central banks, governments and official international organizations on behalf of the Federal Reserve System.",
            "id": "Wv8fBG4Bc3WI8L9MbaO2",
            "link": "https://www.hamilton.edu/news/story/hamilton-nyc-program-tours-federal-reserve-museum",
            "score": 0.10268514747565982,
            "source": "abc15.com",
            "date": "2018-05-24T00:00:00.000Z",
            "tags": [
                "Federal Reserve",
                "New York"
            ]
        }
    ],
    "aggs": {
      "Tag": [{
          "key": "Central bank",
          "count": 878
        }, {
          "key": "Gold",
          "count": 523
        }
      }
}
```

`Total`: Number of results.

`result []`: The array that contains result items.

`aggs` : Facets over the results with count of items for each facet.


### Export

Results can also be exported in XLSX format by performing `GET` requests to:

#### Request

```
curl -X GET
    http://search.api.quantxt.com/reports/puvqrjfhqq/xlsx \
    -H 'X-Api-Key: SECRET_API_KEY'
```

The export output is limited to 5000 rows. All `/search` parameters can be passed here to export the desired slice of the data.



For technical questions please contact <support@quantxt.com>

