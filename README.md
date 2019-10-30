# API Documentation

## Table of content

- [Authentication](#authentication)
- [Dictionaries](#dictionaries)
- [Search](#search)


### Authentication
Authentication to the API is performed via HTTP Basic Auth alongside with providing the user credentials 
in the request payload. 
Basic authentication is used to identify the client application consuming the API. 
If the authentication is performed with the success you will receive access and refresh JWT tokens.

`Basic dGhlaWE7` represents Base64 encoded `CLIENT_USERNAME:CLIENT_PASSWORD` combination used for authenticating the client.

```
curl -X POST \
  http://test.portal.quantxt.com/oauth/token \
  -H 'Authorization: Basic dGhlaWE7' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&username=YOUR_USERNAME&password=YOUR_PASSWORD'
```

If the basic authentication fails, meaning that `CLIENT_USERNAME` or `CLIENT_PASSWORD` are invalid, than the response
will be `HTTP 401`:
```
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```

If the provided username or password is wrong, authentication will fail and return `HTTP 400` and response like:
```
{
    "error": "invalid_grant",
    "error_description": "Bad credentials"
}
```

If the provided data is correct and authentication is successful, than the response will return `HTTP 200` 
and it will contain following data:
```
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzE2Njk4OTcsInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6IjQ5ODA1YjkxLTBhYjItNDlmZS1hMzM1LWJkMDQ0NGJkOTNlNCIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl19.4ltfJD2tOjB5T1_yCgojZDQjYV2oU73dCz0WL6P-ML0",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzIyNzI4OTcsInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6ImJhMjQzMDJiLTQ1NGItNDllMC05Nzc2LTI3NDAyZWUwMTBlYiIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl0sImF0aSI6IjQ5ODA1YjkxLTBhYjItNDlmZS1hMzM1LWJkMDQ0NGJkOTNlNCJ9.MI5tC94XeOqqFMOoKE9kKF-Ek_L5x8LhskYeg33UAlk",
    "expires_in": 1799
}
```
`access_token` is JWT token that will be used in API calls for authorizing the requests and is valid for 30 minutes.

`token_type` indicates that this is `Bearer` (JWT) token

`refresh_token` is JWT token that can be used for refreshing the access token.

`expires_in` indicates for how long access token is valid.

Access token can be refreshed using the `refresh_token` by making a call like:
```
curl -X POST \
  http://test.portal.quantxt.com/oauth/token \
  -H 'Authorization: Basic dGhlaWE7' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=refresh_token&refresh_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzIyNzI4OTcsInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6ImJhMjQzMDJiLTQ1NGItNDllMC05Nzc2LTI3NDAyZWUwMTBlYiIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl0sImF0aSI6IjQ5ODA1YjkxLTBhYjItNDlmZS1hMzM1LWJkMDQ0NGJkOTNlNCJ9.MI5tC94XeOqqFMOoKE9kKF-Ek_L5x8LhskYeg33UAlk'
```

Once you obtain valid access token, any further request to protected API resources must have it included 
in the `Authorization` header prefixed with the token type, which is `Bearer`. Example:
```
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ8.eyJleHAiOjE1NzExNTQzMzksInVzZXJfbmFtZSI6InN1cGVydXNlckBxdWFudHh0LmNvbSIsImp0aSI6ImY4ZTAwNmNiLWVjMTMtNDY3OC1hZWFhLTI0ZmFlMjNlMDA3ZCIsImNsaWVudF9pZCI6InRoZWlhIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl19.nCerUUuqVo2ag3YxNe8aiBTMyfNRtLgvNtF4vbDBgKI'
```

If JWT access token is not provided when protected API endpoints are invoked or the token is expired, 
than the response will be `HTTP 401`:
```
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```

### Dictionaries

New dictionaries can be created in two ways - by providing dictionary entries in the request payload
or by uploading a TSV file containing dictionary entries.
To provide dictionary entries in the request payload, request should look like:
```
curl -X POST \
  http://test.portal.quantxt.com/dictionaries \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
	"name": "Custom dictionary",
	"entries": 
	[
		{
			"key": "TBD",
			"value": "To be done"
		},
		{
			"key": "TBD",
			"value": "To be determined"
		}
	]
}'
```
`JWT_ACCESS_TOKEN` represents the access token gained during the authentication process.

If everything is ok, response will look like:
```
{
    "id": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
    "key": "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
    "name": "Custom dictionary",
    "global": false,
    "entries": [
        {
            "key": "TBD",
            "value": "To be done"
        },
        {
            "key": "TBD",
            "value": "To be determined"
        }
    ]
}
```

Other option is to upload a TSV file containing dictionary entries. If the file is called for example 
`/home/files/custom_dictionary.tsv`, than the upload request should look like:
```
curl -X POST \
  http://test.portal.quantxt.com/dictionaries/upload \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
  -F 'name=Uploaded dictionary' \
  -F file=@/home/files/custom_dictionary.tsv
```
and if everything is ok, than expect the response as in previous case.

To fetch created dictionary perform the request below, by providing the correct dictionary ID:
```
curl -X GET \
 http://test.portal.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
 -H 'Authorization: Bearer JWT_ACCESS_TOKEN'
```
and response will be identical to the one after creating the dictionary.

To update existing dictionary, perform request like:
```
curl -X PUT \
  http://test.portal.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN'
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
and again if everything is ok, expect response as in case of creating or fetching the dictionary.

To delete existing dictionary, simple execute:
```
curl -X DELETE \
  http://test.portal.quantxt.com/dictionaries/58608b0f-a0ff-45d0-b12a-2fb93af1a9ad \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' 
```
and if everything is ok response will be `HTTP 200`.

To fetch all existing dictionaries execute request like:
```
curl -X GET \
  http://test.portal.quantxt.com/dictionaries \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' 
``` 
and response will look like:
```
[
    {
        "id": "58608b1f-a0ff-45d0-b12a-2fb93af1a9ad",
            "key": "user-example-com/58608b1f-a0ff-45d0-b12a-2fb93af1a9ad.csv.gz",
            "name": "Custom dictionary",
            "global": false,
        "entries": []
    }
]
```

### Search

To perform a search by providing the file as a data source, next steps should be performed:
1. First upload a desired file (for example file `/Users/file.pdf`)with data to be searched by performing a request like:
```
curl -X POST \
  http://test.portal.quantxt.com/search/file \
  -H 'Authorization: Bearer JWT_ACCESS_TOKEN' \
  -F file=@/Users/file.pdf
```
and if everything goes ok, response will look like:
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
