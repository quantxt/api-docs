# Theia REST API documentations

## Overview and Definitions

**Theia** is a semantic data extraction tool that can read PDF, HTML, Ms Excel, CSV and plain text documents and extract information in structured format using _dictionaries_. It is essential to understand the utility of _dictionaries_ in extraction context:

### Dictionary

In the simplest case, a dictionary is a list of phrases. Theia searches for every phrase in the dictionary in the input documents and based on the _extraction type_ decides to extract data. 
At a minimum, a dictionary must have a Name and at least one entry, one search phrase.
Users can also assign categories to dictionary entries. 

Searching for dictionary phrases in the content is based on the techniques used in modern full-text search engines. Users can use various text analyzers, synonyms, stop words and fuzziness.


### Extraction Types

Each dictionary can have one of the following 4 extraction types:

1. **None** (or null type): Just search for the phrases and mark them if found. This is mainly used for Tagging documents.
2. **Number**: Search for the phrases AND a number in proximity to the found phrases.
3. **Date**: Search for the phrases AND a date in proximity to the found phrases.
4. **Regex**: Search for the phrases AND a custom regular expression in proximity to the found phrases.

Search phrases and types should appear in reading order, either in a sentence or in a table. By default **Theia** expects the phrase and the type to appear close (but not necessarily next) to each other. Users can configure the allowable gap between dictionary phrases and types using regular expressions.


### Examples

| Input | Dictionary Phrase List | Extraction Type | Output |
|-------|------------|-----------------|--------|
|Revenue was $201.5 million , an increase of 36% year-over-year.| revenue | Number | `201,500,000`|
| <table><tr><td>Activity Date</td><td>Due Date</td><td>Amount</td></tr><tr><td>July 25, 2019</td><td>October 23, 2010</td><td>$117,000</td></tr> </table>| Activity Date | Date | `07/25/2019`|
|Fuel Consumption:  18.1 L/100km| Consumption | Regex ([\d\\.]+) .*?100km| `18.1`|
Gas Consumption is 23.1 L per 100km| Consumption | Regex ([\d\\.]+) .*?100km| `23.1`|
The car consumes 24 litre of gas per 100km| Consumption | Regex ([\d\\.]+) .*?100km| `24`|


In the following we will cover the details of configuring and submitting extraction jobs via our REST API. Extensive end-to-end examples can be found in our Java and Python SDK repositories.


## Table of content

- [Authentication](#authentication)
- [Dictionaries](#dictionaries)
  - [Create a New Dictionary](#create-a-new-dictionary)
  - [Uplaod a TSV Dictionary File](#uplaod-a-tsv-dictionary-file)
  - [Update an Existing Dictionary](#update-an-existing-dictionary)
  - [Delete a Dictionary](#delete-an-existing-dictionary)
  - [List Existing Dictionaries](#list-existing-dictionaries)
- [Data Extraction](#data-extraction)
  - [Extraction Files](#extraction-from-files)
  - [Extraction from Web URLs](#extraction-from-web-urls)
  - [Extraction from Data Streams](#extraction-from-data-streams)
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

### Dictionaries

Dictionaries are a list of phrases used for searching in input documents. Each entry of a dictionary has a `str` and an optional `category`.  Once a phrase is found **Theia** produces an extraction object:

```json
{
	"start": 6188,
	"end": 6196,
	"str": "sales of equipment",
	"line": 309,
	"category": "Sales",
	"dict_name": "Revenue",
	"dict_id": "92e7e423-304a-421c-a612-b6dc4215fd09",
}
```
`str` is the found phrase.
`start`, `end` and `line` are the positions of the found phrase.
`category` (optional) and is only produced if the found phrase was associated with a category.
`dict_name` assigned by user when creating a dictionary
`dict_id` returned by **Theia** once a dictionary is created.


**Theia** uses various strategies for matching on dictionary phrases allowing users to configure the fuzziness of search. User can also provide a list synonyms and stop phrases for the value matching. For example, user can only have "Apple Inc" as one phrase in the dictionary and provide "inc", "corp", "corporation" and "company" as synonyms, allowing you to find all occurrences of "Apple the Company", "Apple Corporations" and "Apple Corp" in the content.


Dictionaries can be created in two ways: 
- By providing dictionary entries in the request payload
- By uploading a TSV file


#### Create A New Dictionary

Create data dictionaries via `/dictionaries` end point as follows:

#### Request

```
curl -X POST \
  http://api.quantxt.com/dictionaries \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
	"name": "M&A",
	"entries": 
	[
		{
			"category": "Take Overs",
			"str": "merger and acquisition"
		},
		{
			"category": "Take Overs",
			"str": "take over"
		}
	]
}'
```


#### Response

```
{
    "id": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
    "name": "M&A",
    "global": false,
    "entries": [
    ]
}
```



#### Uplaod a TSV Dictionary File

Upload TSV data dictionaries via `/dictionaries/upload` endpoint as follows:

#### Request

```
curl -X POST \
  http://api.quantxt.com/dictionaries/upload \
  -H 'X-Api-Key: API_KEY' \
  -F 'name=Uploaded dictionary' \
  -F file=@/home/files/custom_dictionary.tsv
```


#### Fetch an Existing Dictionary:

#### Request

```
curl -X GET \
 http://api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
 -H 'X-Api-Key: API_KEY'
```


#### Update an Existing Dictionary

To update an existing dictionary:

#### Request

```
curl -X PUT \
  http://api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
	"name": "Custom dictionary updated",
	"entries": 
	[
		{
			"category": "TBD",
			"str": "To be done"
		},
		{
			"category": "TBD",
			"str": "To be decided"
		}
	]
}'
```

#### Delete an Existing Dictionary

To delete an existing dictionary:

#### Request

```
curl -X DELETE \
  http://api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'X-Api-Key: API_KEY'
```


#### List Available Dictionaries:

To list all your existing dictionaries:

#### Request

```
curl -X GET \
  http://api.quantxt.com/dictionaries \
  -H 'X-Api-Key: API_KEY'
``` 

#### Response

```
[
    {
        "id": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
        "name": "My dictionary",
        "global": false,
        "entries": []
    }
]
```


### Data Extraction

Data Extraction is the process of identifying search phrases found in the input documents along with extraction types (date, number or regex) and producing structured data. Input documents can be streamed from content files, data APIs or directly from public URLs.


#### Extraction from Files

Files are needed to be uploaded first:

#### Request

```
curl -X POST \
  http://api.quantxt.com/search/file \
  -H 'X-Api-Key: API_KEY' \
  -F file=@/Users/file.pdf
```

PDF, TXT, XLS, XLSX, CSV and HTML formats are supported.

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

`uuid`s along with dictionaries are provided for the exraction engine.

#### Request

```
curl -X POST \
  http://api.quantxt.com/search/new \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
  "title": "My data mining with files and dictionaries",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "chunk" : "PAGE",
  "searchDictionaries": [
        { 
            "vocabId": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
            "vocabValueType": "NUMBER"
        }
  ]
}'
```

`vocabId` (required) id of the dictionary

`title` (optional) but it is highly recommended for distinction between different jobs.

`vocabValueType` (optional) and can be `NUMBER` or `DATETIME` or `REGEX`. If `REGEX` is set user needs to provide the look up regular expression via `phraseMatchingPattern`. The following pattern matches on social securities numbers:

```json
{ 
    "vocabId": "758345h-a0ff-45d0-b12a-2fb93af1a9ad",
    "vocabValueType": "REGEX",
    "phraseMatchingPattern" : "(\d{3}-\d{2}-\d{4})",
}
```

#### Response

```
{
    "id": "puvqrjfhqq",
    "title": "My data mining with files and dictionaries",
    "excludeUttWithoutEntities": true,
    "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
    "searchDictionaries": [
        { 
            "vocabPath": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
            "vocabValueType": "NUMBER"
        }
    ]
}
```

`id` is the extraction job id. You can use it to monitor satus of the job or retrieve the results once completed.


**Request parameters:**

`chunk`
(Optional, string) can be `SENTENCE` or `PAGE` or `NONE`. This will result in splitting input documents into semantic chunks before extraction.



To delete a completed extraction job result set:

#### Request

```
curl -X DELETE \
  http://api.quantxt.com/search/puvqrjfhqq \
  -H 'X-Api-Key: API_KEY'
```


#### Extraction from Web URLs:

Mining can be performed on a list of URLs. All parameters in tagging files are applicable here.

#### Request

```
curl -X POST \
  http://api.quantxt.com/search/new \
  -H 'X-Api-Key: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
   "title": "My search with URLs",
   "urls": ["https://electrek.co/2019/10/29/tesla-model-3-first-electric-car-approved-nyc-yellow-cab/", 
            "https://www.cnbc.com/2019/10/20/electric-car-prices-finally-in-reach-of-millennial-gen-z-buyers.html"]
}'
```


**Theia can process both static and dynamic web pages. However, a number of websites build mechanisms to block internet bots. Theia built-in Web parser is not designed to bypass such blocking mechanisms**


#### Extraction from Data Streams

Extraction data from streams or third party data APIs is supported. Please contact <support@quantxt.com> for details.


#### Status Monitoring

The progress endpoint allows user to check the progress of all extraction jobs:

#### Request

```
curl -X GET
    http://api.quantxt.com/search/progress \
    -H 'X-Api-Key: API_KEY'
```

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


It is also possible to check the progress of a specific job:

#### Request

```
curl -X GET
    http://api.quantxt.com/search/progress/cjaejhvtao \
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
  http://api.quantxt.com/search/puvqrjfhqq \
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
    http://api.quantxt.com/reports/puvqrjfhqq/xlsx \
    -H 'X-Api-Key: API_KEY'
```

The output which is in binary must be saved in ".xlsx" format.

#### Exporting in JSON

#### Request

```
curl -X GET
    http://api.quantxt.com/reports/puvqrjfhqq/json \
    -H 'X-Api-Key: API_KEY'
```

The export output is limited to 5000. All `/search` parameters can be passed here to export the desired slice of the data.


For technical questions please contact <support@quantxt.com>

