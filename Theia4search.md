### Create a new model
**POST** "/init"

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`c` | Model unique identifier | Request Parameter| Yes|
|`file` | Input data file | Request Parameter |Yes|
|`thresh` | Search score threshold. Default is 0. | Request Parameter| No|
|`fuzzy_length` | Minimum word length for fuzzy matching Default is 3.| Request Parameter |No|
|`stoplist` | Input stop word file | Request Parameter |No|
|`conceptlist` | Input entity list file | Request Parameter |No|
|`synonym` | Input synonym file | Request Parameter |No|


### Query an existing Model

**GET** /v2/{id}/match

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|
|`str` | input Search query| Request Parameter | Yes|

### Word frequencies for an existing model


**GET** /v2/{id}/wfl

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|


### Load All existing model files
**GET** /v2/load


### Update an existing model.

**POST** "/{id}"

| Parameter | Description | Type |Required |
| --------- | ---- | -------- | ---------|
|`id` | Model unique identifier | Path Parameter| Yes|
|`file` | Input data file | Request Parameter |Yes|
