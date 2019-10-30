# API Documentation

## Table of content

- [Authentication](#authentication)
- [Data Dictionaries](#data_dictionaries)
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

`expires_in` indicates for how long access token is valid.

Access token can be refreshed by making the following call:
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

If access token is missing or expired the end point will return `HTTP 401`:

```
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```


### Data Dictionaries

Data dictionaries are used for labeling. The engine will search for dictionary values in input utterances and label the matching utterance with the dictionary key:

Dictionary:
M&A => merger and acquisition

Input utterance:
Merger and acquisition report published in 2019.

above will be labeled with `M&A`

New dictionaries can be created in two ways: 
- By providing dictionary entries in the request payload
- By uploading a TSV file containing dictionary entries in format of `key<TAB>value`.

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


To retrieve an availbale dictionary perform the request below, by providing the dictionary ID:

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

Tagging is the process of identifying and labeling entities found in the content. Tagging can be done using unsupervised models as well as using user defined data dictionaries. We will cover both approaches in below.


Upload content .txt, .pdf and .html files for tagging via following call:

#### Request
```
curl -X POST \
  http://test.portal.quantxt.com/search/file \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
  -F file=@/Users/file.pdf
```

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

You can also tag web pages:

#### Request

```
curl -X POST \
  http://search.api.quantxt.com/search/
```





and than to perform a search request, pass previously uploaded file UUID in a request like this:
```
curl -X POST \
  http://test.portal.quantxt.com/search/new \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
  -d '{
	"title": "Microsoft",
	"files": ["b351283c-330c-418b-8fb7-44cf3c7a09d5"]
}'
```
and if everything is ok, response should be `HTTP 201`:
```
{
    "index": "puvqrjfhqq"
}
```
and `index` represent the unique identification for previously performed search. It is optional to provide `title` 
but it is recommended for easier distinction between searches.

To fetch the data from performed search, simply execute request like:
```
curl -X GET \
  http://test.portal.quantxt.com/search/puvqrjfhqq \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
```
and response should be in form:
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
Results can also be exported in XLSX or PDF format by simply performing `GET` requests to:
`http://test.portal.quantxt.com/reports/lcjjoghfwk/xlsx` or `http://test.portal.quantxt.com/reports/lcjjoghfwk/pdf` respectively.

To perform a search on files using dictionaries, all that is required is to provide a specific dictionaries keys. 
Dictionary `key` can be found in a create dictionary response like this [here](#dictionaries). 
Other way is to [fetch all dictionaries](#dictionaries) and than find the key properties of the dictionaries 
that should be used for search. So, using identifiers of the files and dictionaries we previously created, 
request should look like this:
```
curl -X POST \
  http://test.portal.quantxt.com/search/new \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
  -d '{
	"title": "My search with files and dictionaries",
	"files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
	"dictionaries": ["user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz"]
}'
```
and if everything goes alright, response should be in exactly the same format as in previous search case.

To perform a search on files using dictionaries and types as well, we need to append a type parameter to the dictionary,
 separated by the `||`.
In that case, request should look like:
```
curl -X POST \
  http://test.portal.quantxt.com/search/new \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
  -d '{
	"title": "My search with files, dictionaries and types",
	"files": ["c351283c-330c-418b-8fb7-44cf3c7a09d5"],
	"dictionaries": ["user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz||DOUBLE"]
}'
```
and again, if everything is ok, response will be the same as in previous search cases.
Supported types at the moment are:
`SHORT, INT, LONG, FLOAT, DOUBLE, STRING, KEYWORD, BOOL, DATETIME, NOUN, VERB, PERCENT, MONEY, NONE`.

To delete the search, simply perform delete request like:
```
curl -X DELETE \
  http://test.api.quantxt.com/search/puvqrjfhqq \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
```
where `puvqrjfhqq` represents the search `index`. If everything is ok, `HTTP 200` will be returned.
