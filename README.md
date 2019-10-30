# Theia Search API Documentation

## Table of content

- [Authentication](#authentication)
- [Data Dictionaries](#data-dictionaries)
- [Tagging](#tagging)
- [Search](#search)
- [Export](#export)


### Authentication
Authentication to the API is performed via HTTP Basic Auth by providing the user credentials 
in the request payload. 
If the authentication is performed successfully you will receive access and refresh tokens.

`Basic dGhlaWE7` represents Base64 encoded `CLIENT_USERNAME:CLIENT_PASSWORD` combination used for authenticating the client.

```
curl -X POST \
  http://search.api.quantxt.com/oauth/token \
  -H 'Authorization: Basic dGhlaWE7' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&username=YOUR_USERNAME&password=YOUR_PASSWORD'
```

If the provided username or password is wrong, authentication will fail and return `HTTP 400`:
```
{
    "error": "invalid_grant",
    "error_description": "Bad credentials"
}
```

If authentication is successful, it will return `HTTP 200` 
with `access_token` and `refresh_toekn` in the body of the response:
```
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzE2Njk4OTcsInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6IjQ5ODA1YjkxLTBhYjItNDlmZS1hMzM1LWJkMDQ0NGJkOTNlNCIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl19.4ltfJD2tOjB5T1_yCgojZDQjYV2oU73dCz0WL6P-ML0",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzIyNzI4OTcsInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6ImJhMjQzMDJiLTQ1NGItNDllMC05Nzc2LTI3NDAyZWUwMTBlYiIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl0sImF0aSI6IjQ5ODA1YjkxLTBhYjItNDlmZS1hMzM1LWJkMDQ0NGJkOTNlNCJ9.MI5tC94XeOqqFMOoKE9kKF-Ek_L5x8LhskYeg33UAlk",
    "expires_in": 1799
}
```
`access_token` should be used in all API calls and must be refreshed every `expires_in` minutes.

`token_type` indicates that this is `Bearer` (JWT) token

`refresh_token` is the token to be used for refreshing the `access_token`.

`expires_in` indicates for how long the access token is valid.

The access token can be refreshed by making the following call:
```
curl -X POST \
  http://search.api.quantxt.com/oauth/token \
  -H 'Authorization: Basic dGhlaWE7' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=refresh_token&refresh_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzIyNzI4OTcsInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6ImJhMjQzMDJiLTQ1NGItNDllMC05Nzc2LTI3NDAyZWUwMTBlYiIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl0sImF0aSI6IjQ5ODA1YjkxLTBhYjItNDlmZS1hMzM1LWJkMDQ0NGJkOTNlNCJ9.MI5tC94XeOqqFMOoKE9kKF-Ek_L5x8LhskYeg33UAlk'
```

Calls to all other API end points must include the `access_token` in their header:

```
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzExNTQzMzksInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6ImY4ZTAwNmNiLWVjMTMtNDY3OC1hZWFhLTI0ZmFlMjNlMDA3ZCIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl19.nCerUUuqVo2ag3YxNe8aiBTMyfNRtLgvNtF4vbDBgKI'
```

If the access token is missing or expired the endpoint will return `HTTP 401`:

```
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```


### Data Dictionaries

Data dictionaries are used for labeling. **Theia** will search for dictionary values in input utterances and label the matching utterance with the dictionary key:

Dictionary:
M&A => merger and acquisition

Input utterance:
Merger and acquisition report published in 2019.

above will be labeled with `M&A`

New dictionaries can be created in two ways: 
- By providing dictionary entries in the request payload
- By uploading a TSV file containing dictionary entries in the format of `key<TAB>value`.

Create data dictionaries via `/dictionaries` end point as follows:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/dictionaries \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
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

`ACCESS_TOKEN` represents the access token gained during the authentication process.

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

Upload TSV data dictionaries via `/dictionaries/upload` end point as follows:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/dictionaries/upload \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
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
 -H 'Authorization: Bearer ACCESS_TOKEN'
```

To update an existing dictionary:

#### Request

```
curl -X PUT \
  http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'Authorization: Bearer ACCESS_TOKEN'
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

To delete an existing dictionary:

#### Request

```
curl -X DELETE \
  http://search.api.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'Authorization: Bearer ACCESS_TOKEN' 
```


To list all existing dictionaries:

#### Request

```
curl -X GET \
  http://search.api.quantxt.com/dictionaries \
  -H 'Authorization: Bearer ACCESS_TOKEN' 
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

Tagging is the process of identifying and labeling entities found in the content. Tagging can be done using unsupervised models as well as using user-defined data dictionaries. We will cover both approaches in below.


#### Tagging content files

First, upload all content files for tagging via the following call:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/file \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
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
  -H 'Authorization: Bearer ACCESS_TOKEN' \
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

`index` represents the unique identification for the container that holds labeled data. 


Supported parameters:

`get_phrases`
(Optional, boolean) if `true` it will use built-in Entity tagging engine.

`excludeUttWithoutEntities`
(Optional, boolean) if `true` the output only includes utterances that have at least one tag from the input dictionaries.

`runSentenceDetect`
(Optional, boolean) if `true` it will break the input content unit into semantic sentences and run tagging at the sentence level.


To delete a data container:

#### Request

```
curl -X DELETE \
  http://search.api.quantxt.com/search/puvqrjfhqq \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
```


#### Tagging Web pages:

Tagging can be performed on a list of URLs. All parameters in tagging files are applicable here.

#### Request


```
curl -X POST \
  http://search.quantxt.com/search/new \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
  -d '{
   "title": "My search with URLs",
   "urls": ["https://electrek.co/2019/10/29/tesla-model-3-first-electric-car-approved-nyc-yellow-cab/", 
            "https://www.cnbc.com/2019/10/20/electric-car-prices-finally-in-reach-of-millennial-gen-z-buyers.html"]
}'
```


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
  -H 'Authorization: Bearer ACCESS_TOKEN' \
  -d '{
  "title": "My search with dictionaries and types",
  "files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
  "dictionaries": ["user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz||INT"]
}'
```


### Search


To fetch the data from performed search, simply execute request like:

#### Request

```
curl -X GET \
  http://search.api.quantxt.com/search/puvqrjfhqq \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
```


#### Response

```
{
    "Total": 2610,
    "results": [
        {
            "title": "Microsoft has pulled its plans to build a office in SF, the city confirmed on Friday.",
            "id": "Wv8fBG4Bc3WI8L9MbaO2",
            "link": "https://www.abc15.com/news/microsoft",
            "score": 0.10268514747565982,
            "source": "abc15.com",
            "date": "2018-05-24T00:00:00.000Z",
            "tags": [
                "City",
                "Microsoft"
            ]
        }
    ],
    "aggs": {},
    "meta": {
        "title": "Microsoft",
        "sub_title": "",
        "source": [
            "news"
        ],
        "fields": [],
        "dateUpdated": "2018-05-24T00:00:00.000Z",
        "numResults": 2610,
        "progress": 100,
        "num_doc_new": 286,
        "num_new_res_after": 2610,
        "took": 17272,
        "num_new_res_before": 7145,
        "index": "puvqrjfhqq",
        "mode": 2,
        "thresh": 0.3,
        "alignScoreL": 0.1,
        "search_terms": [
            "microsoft"
        ]
    }
}
```



### Export

Results can also be exported in XLSX format by performing `GET` requests to:

```
curl -X GET
    http://search.api.quantxt.com/reports/puvqrjfhqq/xlsx \
    -H 'Authorization: Bearer ACCESS_TOKEN'
```

The export output is limited to 5000 rows. All `/search` parameters can be passed here to export the desired slice of the data.