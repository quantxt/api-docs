## Theia API documentations; Overview and Definitions


**Theia** is a semantic data extraction tool. Theia can read PDF, HTML, Ms Excel, CSV and plain text documents and extract information in structured format using _dictionaries_. It is important to understand dictionaries and their utility in extraction context:

### Dictionary

In the simplest case, a dictionary is a list of phrases. Theia search for every phrase in the dictionary in the input documents and based on the extraction type results decide to extract information. 
At a minimum, a dictionary must have a Name and at least one entry (one search phrase).
Users can also assign categories to dictionary entries. 

Searching for dictionary phrases in the content is based on the techniques used in modern search engines. Users can use various text analyzers, synonyms, stop words and fuzziness. All of this will be explained in the examples.


### Extraction Types

Each dictionary can have one of the following 4 extraction types:

1. **None** (or null type): Just search for the phrases. This is mainly used for Tagging documents.
2. **Number**: Search for the phrases AND a number in proximity to the phrase.
3. **Date**: Search for the phrases AND a date in proximity to the phrase.
4. **Regex**: Search for the phrases AND a custom regular expression in proximity to the phrase.

Dictionary phrases and Numbers or dates or regular expressions must appear in semantic order, either in a sentence or in a table. By default the phrase and the type should appear close (but not necessarily next) to each other. Users can configure the allowable gap between dictionary phrases and types using regular expressions.


### Examples

| Input | Dictionary Phrase List | Extraction Type | Output |
|-------|------------|-----------------|--------|
|Subscription revenue was $201.5 million , an increase of 36% year-over-year.| Subscription revenue | Number | `201,500,000`|
| <table><tr><td>Activity Date</td><td>Due Date</td><td>Amount</td></tr><tr><td>July 25, 2019</td><td>October 23, 2010</td><td>$117,000</td></tr> </table>| Activity Date | Date | `07/25/2019`|


In the following we cover details of configuring and submitting extraction jobs via a RESTful API. 


## Table of content

- [Authentication](#authentication)
- [Dictionaries](#dictionaries)
  - [Create a New Dictionary](#create-a-new-dictionary)
  - [Uplaod a TSV Dictionary File](#uplaod-a-tsv-dictionary-file)
  - [Update an Existing Dictionary](#update-an-existing-dictionary)
  - [Delete a Dictionary](#delete-a-dictionary)
  - [List Available Dictionaries](#list-available-dictionaries)
- [Extraction and Mining](#extraction-and-mining)
  - [Mining Content Files](#mining-content-files)
  - [Mining Web URLs](#mining-web-urls)
  - [Mining Data Streams](#mining-data-streams)
  - [Extracting Typed Entities](#extracting-typed-entities)
  - [Status Monitoring](#status-monitoring)
- [Searching in the Results](#searching-in-the-results)
- [Exporting the Results](#exporting-the-results)
  -[Exporting in Excel Format](#exporting-in-excel-format)
  -[Exporting in JSON](#exporting-in-json)


### Authentication

All API calls to protected end points must include a header `X-Api-Key` with a valid API key:

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


"Net Revenue" is a Typed Entity:

```
"Apple reported of net revenue of $53.3 billion in 2018."
```

"Net Revenue" is not a Typed Entity:

```
"Tim Cook will go over net revenue of Apple in the upcoming conference call."
```


### Dictionaries

Dictionaries are simply a list of phrases that are. Each item of a dictionary has a `str` and a `category`. The `str` is a search phrase and the `category` is the actual word or phrase that represents the entity.

**Theia** scans all input utterances for every word or phrase in the dictionary and, if found, map it to the associated value:

Dictionary (one item):
```merger and acquisition => M&A```

Input utterance:

>Merger and acquisitions report published in 2019.

The above will be labeled with `M&A`

**Theia** uses various strategies for matching on dictionary phrases allowing users to configure the fuzziness of search. User can also provide a list synonyms and stop phrases for the value matching. For example, user can only have "Apple Inc" as one phrase in the dictionary and provide "inc", "corp", "corporation" and "company" as synonyms, allowing you to find all occurrences of "Apple the Company", "Apple Corporations" and "Apple Corp" in the content.


Entity dictionaries can be created in two ways: 
- By providing dictionary entries in the request payload
- By uploading a TSV file containing dictionary entries in the format of `key<TAB>value`.


#### Create A New Dictionary

Create data dictionaries via `/dictionaries` end point as follows:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/dictionaries \
  -H 'X-Api-Key: API_KEY' \
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



#### Uplaod a TSV Dictionary File

Upload TSV data dictionaries via `/dictionaries/upload` endpoint as follows:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/dictionaries/upload \
  -H 'X-Api-Key: API_KEY' \
  -F 'name=Uploaded dictionary' \
  -F file=@/home/files/custom_dictionary.tsv
```


To retrieve an available dictionary perform the request below, by providing the dictionary ID:

#### Request

```
curl -X GET \
 http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
 -H 'X-Api-Key: API_KEY'
```


#### Update an Existing Dictionary

To update an existing dictionary:

#### Request

```
curl -X PUT \
  http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
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
  -H 'X-Api-Key: API_KEY'
```


#### List Available Dictionaries:

To list all your existing dictionaries:

#### Request

```
curl -X GET \
  http://search.api.quantxt.com/dictionaries \
  -H 'X-Api-Key: API_KEY'
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


### Extraction and Mining

Data Extraction and Mining is the process of identifying entities found in the unstructured content using entity dictionaries and formatting it into structured format. Unstructured data can be streamed from content files, data APIs or directly from public URLs.


#### Mining Content Files

First, upload all content files for tagging via the following call:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/file \
  -H 'X-Api-Key: API_KEY' \
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

Then you can mine data via dictionaries:

#### Request

```
curl -X POST \
  http://search.quantxt.com/search/new \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
  "title": "My data mining with files and dictionaries",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "searchDictionaries": [
        { 
            "vocabPath": "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
            "vocabValueType": "NONE"
        }
  ]
}'
```

`title` is optional but it is highly recommended for easier distinction between different tagging jobs.

`vocabValueType` can have one of the following values: `NONE`, `STRING`, `DOUBLE`, `DATETIME`, 

**There is no limit on the number of files and dictionaries that can be processed via `/new` end-point.**

#### Response

```
{
    "index": "puvqrjfhqq",
    "title": "My data mining with files and dictionaries",
    "get_phrases": true,
    "maxTokenPerUtt": 35,
    "minTokenPerUtt": 6,
    "excludeUttWithoutEntities": false,
    "stitle": null,
    "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
    "searchDictionaries": [
        { 
            "vocabPath": "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
            "vocabValueType": "NONE"
        }
    ]
}
```

`index` represents the unique identification for the container that holds output labeled data. 


**Request parameters:**

`chunk`
(Optional, string) can be `SENTENCE` or `PARAGRAPH` or `NONE`. This will result in splitting data into semantic chunks before processing. For example, this allow user to split the content of an article in semantic sentences and apply entity dictionaries at sentence level.

`excludeUttWithoutEntities`
(Optional, boolean) if `true` the output only includes chunks that have at least one label from the input dictionaries.

`get_phrases`
 (Optional, boolean) if `true` auto tagging will be performed.
 
`minTokenPerUtt` (Optional, int)
 
`maxTokenPerUtt` (Optional, int) 

`stitle` (Optional, string) Override command.

To delete a data container:

#### Request

```
curl -X DELETE \
  http://search.api.quantxt.com/search/puvqrjfhqq \
  -H 'X-Api-Key: API_KEY'
```


#### Mining Web URLs:

Mining can be performed on a list of URLs. All parameters in tagging files are applicable here.

#### Request

```
curl -X POST \
  http://search.quantxt.com/search/new \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
   "title": "My search with URLs",
   "urls": ["https://electrek.co/2019/10/29/tesla-model-3-first-electric-car-approved-nyc-yellow-cab/", 
            "https://www.cnbc.com/2019/10/20/electric-car-prices-finally-in-reach-of-millennial-gen-z-buyers.html"]
}'
```


**Theia can process both static and dynamic web pages. However, a number of websites build mechanisms to block internet bots. Theia built-in Web parser is not designed to bypass such blocking mechanisms**

#### Mining Data Streams

Tagging data from data streams such as third party APIs is supported. Please contact <support@quantxt.com> for details.


#### Extracting Typed Entities

Entity dictionaries allow the user to quickly search and label thousands of phrases in unstructured content. There are cases when users want to label a keyword or phrase as an entity only if it is associated with a value. For example:

A "release => "Manufactured" dictionary item will label both of the following utterances:

> The first automobile in the US **released** by Ford
> The first automobile in the US **released** in 1908 by Ford

However, if the user is looking for only release year of the car makers the first utterance won't have much use for him. He can only label the second utterance using a **Typed Entities**.

Supported types for associated entities are:

`REGEX`, `DOUBLE`, `DATETIME`

types are passed via `vocabValueType` parameter.

If a Type is set, a dictionary item will be extracted only if a type is found in its close proximity.
In the above example, user can set *vocabValueType* in the request to `DOUBLE` to identify **1908** only if it is associated with the entity **released**

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/new \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
  "title": "My search with dictionaries and types",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "searchDictionaries": [
    {
      "vocabPath" : "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
      "vocabValueType": "DOUBLE"
    }
  ]
}'
```

`vocabPath` (Required): The path to entity dictionary. Path is returned either after creation of a new dictionary or via listing existing dictionaries.

`vocabValueType` (Optional): If set, the engine will extract only entities that are associated with a entity of this type.

To make Theia extract entities using a regular expression *vocabValueType* needs to be set to `REGEX`. You also need to provide the actual regular expression as well as the capturing groups:


#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/new \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
  "title": "My search with dictionaries and types",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "searchDictionaries": [
    {
      "vocabPath" : "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
      "vocabValueType": "REGEX",
      "phraseMatchingPattern": "([A-Z]{3})\\-([A-Z0-9]+)(\\d)",
      "phraseMatchingGroups": ["1" , "3"]
    }
  ]
}'
```

#### Status Monitoring

The Progress endpoint allows user to check the progress of a submitted data mining job:

#### Request

```
curl -X GET
    http://search.api.quantxt.com/search/progress \
    -H 'X-Api-Key: API_KEY'
```

The search result is a array of active data mining jobs:

```
[
    {
        "index": "cjaejhvtao",
        "progress": 36,
        "progress_msg": "Collecting data..."
    }
]

```

`index` Unique ID of the running job

`progress`  Progress in %. a number between 0 to 100.

`progress_msg` (Optional) Progress message.


It is also possible to check the progress of a specific data mining job:

#### Request

```
curl -X GET
    http://search.api.quantxt.com/search/progress/cjaejhvtao \
    -H 'X-Api-Key: API_KEY'
```

#### Response

```
{
    "index": "cjaejhvtao",
    "progress": 36,
    "progress_msg": "Collecting data..."
}
```


### Searching in the Results


The Search endpoint allows user to run full-text and [faceted search](https://en.wikipedia.org/wiki/Faceted_search) in the extracted data.

#### Request

```
curl -X GET \
  http://search.api.quantxt.com/search/puvqrjfhqq \
  -H 'X-Api-Key: API_KEY'
```

**Request parameters:**

`q`
(Optional, string) Search query that goes against the main content `title` field. It supports boolean `OR`, `AND` and `NOT` parameters.

`f`
(Optional, string) Query filters and must be used in pairs. Filters are created for each input dictionary. For example to include results that have one or more label from `Vehicle` dictionary the request should look like: `&f=Vehicle&f=*`. To include results that are labeled with `Ford` or `BMW` from the `Vehicle` dictionary, the request would be `&f=Vehicle&f=BMW&f=Vehicle&f=Ford`

`from`
(Optional, int) Offset for paging results. Dafaults to 0. Each page contain 20 items.

`size`
(Optional, int) Number of results to return. Maximum is 200.


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


### Exporting the Results

Results can also be exported in XLSX format by performing `GET` requests to:

#### Exporting in Excel Format

#### Request

```
curl -X GET
    http://search.api.quantxt.com/reports/puvqrjfhqq/xlsx \
    -H 'X-Api-Key: API_KEY'
```

The output which is in binary must be saved in ".xlsx" format.

#### Exporting in JSON

#### Request

```
curl -X GET
    http://search.api.quantxt.com/reports/puvqrjfhqq/json \
    -H 'X-Api-Key: API_KEY'
```

The export output is limited to 5000. All `/search` parameters can be passed here to export the desired slice of the data.


For technical questions please contact <support@quantxt.com>

