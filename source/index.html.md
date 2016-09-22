---
title: Proband Connect API Reference

language_tabs:
  - shell
toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:

search: true
---

# Introduction

This is the documentation for the Proband Connect REST API.

# Authentication

`/api-token-auth/`

> To get a valid session id, use the following code

```shell

curl 'https://probandapp.com/connect/api-token-auth/' \
      -H 'Content-Type: application/json' \
     --data-binary '{"username": "jeffmax",
                     "password": "asecret",
                     "device":   "unique device id"}' \
     --compressed

```

> Make sure to replace `unique device id` with a unique device id associated with your user.
> The above command returns JSON structured like this:

```json
    {"token":"f8f6ba3b39e83568ea17135d3a836d93e84caf93"}
```

The Proband Connect API requires that you first obtain a session ID using a valid `user`, `password`, and `device_id`. 

### HTTP Request

`POST http://probandapp.com/connect/api-token-auth/`

#### JSON POST Payload

Parameter  | Description
---------  | -----------
user       | Username
pssword    | Password
device     | Unique device/activity ID

<aside class="notice">
 Each user must be configured with a device/activity ID for each unique device or role that will be accessing the API. This can be done through the Django Admin interface (it is called a DeviceToken in the admin). You can either create one associated to the user with a preset ID or leave the ID empty. If left empty, the first time the user authenticates and reports an ID, that ID will be assigned. From then on, only that ID will work.
</aside>

Proband Connect requires the returned seesion key to be included in all API requests to the server in a header that looks like the following:

     Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93

# Pedigrees

`/pedigrees`

## GET

```shell

curl 'https://probandapp.com/connect/pedigrees/' \
  -H 'Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93'

```
> This request will return a JSON list of objects containing the Pedigree UUID and list of groups the pedigree belongs to:

```json
 
[ 
  { "uuid": "1aksjdfl2j3q234lskdjfadfjhj98", 
    "groups":[ {"name": "Genetics",
                "id"  : 1},
             ]
  },
]

```

### HTTP Request

Return a list of pedigrees the authenticated user has access to:

`GET https://probandapp.com/connect/pedigrees/`

## POST

```shell

curl 'https://probandapp.com/connect/pedigrees/' \
      -H 'Content-Type: application/json' \
      -H 'Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93' \
     --data-binary '[{"force": False,
                     "version" : 2,
                     "uuid": "1aksjdfl2j3q234lskdjfadfjhj98",
                     "content": "<pedigree></pedigree>"},]' \
     --compressed

```
> This will return a list JSON objects listing which UUID updates failed, which succeeded, and which were deleted from the server (so could not be updated):

```json
 
{
  "failed":  [ { "uuid": "1aksjdfl2j3q234lskdjfadfjhj98", 
                 "error": "UUID Unknown" },
  ],
  "success": [ { "uuid": "1aksjdfl2j3q234lskdjfadfjhj98",
                 "version": 3,
                 "hash": "MD5 HASH of content "},
  ],
  "deleted": ["asdkjfklasdjflkasjdflkjasdflksdjafkljsd"]
}

```

This endpoint allows you to create/update multiple pedigrees at once.

### HTTP Request

`POST https://probandapp.com/connect/pedigrees/`

#### JSON POST Payload

A list of JSON objects, each with the follow properties:

Parameter | Type    | Description
--------- | ------  | -----
force     | Boolean | Whether or not to force write over a newer pedigree version
content   | String  | XML pedigree
uuid      | String  | Pedigree UUID
version   | Integer | Pedigree version


# Pedigree

`/pedigree/`

## GET

```shell

curl 'https://probandapp.com/connect/pedigree/1aksjdfl2j3q234lskdjfadfjhj98/?ifNewer=1' \
     -H 'Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93' 
```

> This will return a JSON object representing the pedigree with UUID `1aksjdfl2j3q234lskdjfadfjhj98` if there is a version on the server newer than 1. The `ifNewer` parameter is optional. 

```json
 { 
        "update": True,  #This is only present when ifNewer is used, True if a newer pedigree is available
        "version" : 2,
        "uuid": "aksjdfl2j3q234lskdjfadfjhj98", 
        "hash: "<MD5 Hash of Pedigree>"",
        "content":"<pedigree></pedigree>", # Not present if "update" is False
        "user": "probanduser", 
        "device": "<device/activity ID>",
        "published": False, # Whether this Pedigree is marked as published and has PDF on File
        "groups":[{'name':"genetics", "id":1},]
}
```

This endpoint is used to retrieve a pedigree using its UUID. 

### HTTP Request

`GET  https://probandapp.com/connect/pedigree/<UUID>/`
    
#### GET Query Params

Parameter | Type    | Description
--------- | ------  | -----
ifNewer   | Integer | Specify the version you currently have- if the server does not have a newer version, it won't return the pedigree. Optional.

## POST

```shell

curl 'https://probandapp.com/connect/pedigree/' \
      -H 'Content-Type: application/json' \
      -H 'Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93' \
     --data-binary '{"uuid": "1aksjdfl2j3q234lskdjfadfjhj98",
                     "content": "<pedigree></pedigree>"},]',
                     "version": 0}'
     --compressed
```

> This will create a new pedigree with the supplied UUID. It will return a JSON object representing the status of the new pedigree:

```json
{ 
        "uuid": "aksjdfl2j3q234lskdjfadfjhj98", 
        "hash: "<MD5 Hash of Pedigree>",
        "published: False, # Whether this Pedigree is marked as published and has PDF on File
        "version": 1,
}
```

This endpoint is used to create a new pedigree. 

### HTTP Request

`POST  https://probandapp.com/connect/pedigree/`

### POST JSON Payload
A JSON object with the following properties

Parameter | Type    | Description
--------- | ------  | -----
content   | String  | XML pedigree
uuid      | String  | Pedigree UUID
version   | Integer | Pedigree version, should only ever be 0
published | String  | Base64 encoded PDF of pedigree. Optional. Only use if you want to mark pedigree as published and make a PDF accessible to 3rd parties

## PUT

```shell

curl 'https://probandapp.com/connect/pedigree/1aksjdfl2j3q234lskdjfadfjhj9' \
      -H 'Content-Type: application/json' \
      -H 'Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93' \
      -X PUT \
      --data-binary '{"content": "<pedigree></pedigree>"},]',
                      "version": 3,
                      "force": false }'
     --compressed
```

> This will update the pedigree specified in the URL. It will return a JSON object representing the current status of the pedigree:

```json
{ 
        "uuid": "aksjdfl2j3q234lskdjfadfjhj98", 
        "hash: "<MD5 Hash of Pedigree>",
        "published: False, # Whether this Pedigree is marked as published and has PDF on File
        "version": 4,
}
```

This endpoint is used to update an existing pedigree. 

### HTTP Request

`PUT  https://probandapp.com/connect/pedigree/1aksjdfl2j3q234lskdjfadfjhj9/`

### PUT JSON Payload

A JSON object with the following properties:

Parameter | Type    | Description
--------- | ------  | -----
content   | String  | XML pedigree
uuid      | String  | Pedigree UUID
version   | Integer | Pedigree version, should only ever be 0
published | String  | Base64 encoded PDF of pedigree. Optional. Only use if you want to mark pedigree as published and make a PDF accessible to 3rd parties
force     | Boolean | Whether to force write this pedigree if there is a conflict. Optional.

## DELETE

```shell

curl 'https://probandapp.com/connect/pedigree/1aksjdfl2j3q234lskdjfadfjhj9/' \
  -X DELETE \
  -H 'Authorization: f8f6ba3b39e83568ea17135d3a836d93e84caf93'
```

> This command will delete the pedigree specified in the URL. It will return a JSON object confirming the action:

```json
{ 
        "uuid": "1aksjdfl2j3q234lskdjfadfjhj98", 
        "deleted": true,
}
```

This endpoint is used to delete an existing pedigree from the server.

### HTTP Request

`DELETE  https://probandapp.com/connect/pedigree/1aksjdfl2j3q234lskdjfadfjhj9/`

# Published

Find out if there is a published version of a pedigree

# UUIDS

Get UUIDS for creating new pedigrees

# PDF

Get a published PDF for a pedigree

# Browse

Search for pedigrees by name or diagnoses
