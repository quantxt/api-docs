Theia is a stanalone, single node full text search engine. It takes the input content in form of csv, indexes the content and provide an end point to search through the content.
&nbsp;

Unlike other search engines, Theia does not require configuring tokenizers and/or text analyzers for exact or fuzzy matching as it is all handled automatically and optimized on the content.
&nbsp;

Each row in the input CSV is considered one instance. Each row must have an integer ID in first column and some searchable content (aka text) in column[2] or after. column[1] can not contain searchable content.

### Create a new model
**POST** `/init`

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`c` | Model unique identifier | Request Parameter| Yes|
|`file` | Input CSV data file | Request Parameter |Yes|
|`config` | Search configuration in string Json format |No|
|`thresh` | Search score threshold. Default is 0. | Request Parameter|No|
|`fuzzy_length` | Minimum word length for fuzzy matching Default is 3.| Request Parameter |No|
|`stoplist` | Input stop word file | Request Parameter |No|
|`synonym` | Input synonym file | Request Parameter |No|

&nbsp;
&nbsp;

Takes a CSV file as input and create a full text search engine on columns set by `search_cols`
User is responsible to provide a unique id (`c`) for the anticipated search engine.
&nbsp;

First row of the inout CSV is skipped
Column 0 must be the id or row number and is always returned in the search response
Column 1 is reserved and can not be searchable.

### Search Configuration ###

You can configure multiple strategies for searching the content. Each strategy reads content from one of the inout CSV columns and index them using
the defined settings. Each strategy has 5 parameters:


| Parameter | Description | Possible Values |Required |
| --------- | ---- | -------- | ---------|
|`input_col` | 0-based column index in input CSV | Integer >=2| Yes|
|`mode` | Full-Text search mode| `SPAN` `PARTIAL_SPAN` `PARTIAL_FUZZY_SPAN` | Yes|
|`analyzType` | Text processing method| `STEM` `SIMPLE` `WHITESPACE` | Yes|
|`thresh` | Search score threshold. Default is 0. | Float >=0|No|
|`fuzzy_length` | Minimum word length for fuzzy matching Default is 3.| Integer >=0 |No|


#### Mode ####

`SPAN` : Match on all keywords  <br />
`PARTIAL_SPAN` : Match on at least one keyword  <br />
`PARTIAL_FUZZY_SPAN` : Fuzzy-Match on at least one keyword  <br />


### Query an existing Model

**GET** `/v2/{id}/match`

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|
|`str` | input Search query| Request Parameter | Yes|

&nbsp;
&nbsp;


### Load All existing model files
**GET** `/v2/load`

&nbsp;
&nbsp;

### Update an existing model.

**POST** `/{id}`

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|
|`file` | Input data file | Request Parameter |Yes|

&nbsp;
&nbsp;

### Delete an existing model.

**DELETE** `/{id}`

&nbsp;
&nbsp;

### Healthchecks

#### Service status
**GET** `/v1/healthcheck`

#### List of live models
**GET** `/v2/insights`




