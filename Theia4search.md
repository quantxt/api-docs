### Create a new model
**POST** `/init`

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`c` | Model unique identifier | Request Parameter| Yes|
|`file` | Input CSV data file | Request Parameter |Yes|
|`search_cols` | Comma separated columns from the CSV input that needs to be searchable | Request Parameter|Yes|
|`thresh` | Search score threshold. Default is 0. | Request Parameter| No|
|`fuzzy_length` | Minimum word length for fuzzy matching Default is 3.| Request Parameter |No|
|`stoplist` | Input stop word file | Request Parameter |No|
|`conceptlist` | Input entity list file | Request Parameter |No|
|`synonym` | Input synonym file | Request Parameter |No|

&nbsp;
&nbsp;

Takes a CSV file as input and create a full text search engine on columns set by `search_cols`
User is responsible to provide a unique id (`c`) for the anticipated search engine.
&nbsp;

First row of the inout CSV is skipped
Column 0 must be the id or row number and is always returned in the search response
Column 1 is reserved and can not be searchable.


### Query an existing Model

**GET** `/v2/{id}/match`

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|
|`str` | input Search query| Request Parameter | Yes|

&nbsp;
&nbsp;

### Word frequencies for an existing model


**GET** `/v2/{id}/wfl`

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|

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




