
# Syncano library developer starter

### !!! This guide refers to the early alpha version of syncano v4. !!!

[Python Examples](https://github.com/Syncano/syncano-python/tree/release/4.0)

### First interaction with the API
Current staging infrastructure is located at:
https://syncanotest1-env.elasticbeanstalk.com/
But it also has an dns alias to:
https://v4.hydraengine.com/

Use following links to see the docs, or to navigate to the first version of api:

- readme.io [docs](http://syncano.readme.io/v4.0/docs/api-explorer-usage)
- api [/v1/](https://syncanotest1-env.elasticbeanstalk.com/v1/) - where is the first version of the api

#### Note:

Sometimes in the readme.io documentation you can see a url api.syncano.io as backend url. It's not ready yet. Use https://v4.hydraengine.com or https://syncanotest1-env.elasticbeanstalk.com/
Those sites don't have a valid certificats (self signed), so you would probably have to ignore it for now.


### How I register new account and obtain the proper key?
New account can be registered at:
https://syncanotest1-env.elasticbeanstalk.com/v1/account/register/

Then you need to authorize to obtain the api key:
https://syncanotest1-env.elasticbeanstalk.com/v1/account/auth/

You will obtain something like this:
```
{
    "id": 10,
    "email": "justyna.ilczuk@gmail.com",
    "first_name": "Justyna",
    "last_name": "Ilczuk",
    "account_key": "my-very-secret-key"
}
```

Account key will be later used in authentication and authorization.

A good start for developing a library might be playing interactively with api in our docs available at:
https://syncanotest1-env.elasticbeanstalk.com/docs/

If you add api_key in right upper corner, you would be able to use api as authorized user.

### How authentication & authorization works?

There are few options there:

- add authorization header
```
curl -H "AUTHORIZATION: token my-api-key" -k https://syncanotest1-env.elasticbeanstalk.com/v1/instances/my-instance/
```
- add apikey header
```
curl -H "X-API-KEY: my-api-key" -k https://syncanotest1-env.elasticbeanstalk.com/v1/instances/my-instance/
```
- add query parameter to request
```
https://syncanotest1-env.elasticbeanstalk.com/v1/instances/?api_key=my-api-key
```

### How paging works?
Example for codebox traces:

We use direction 1, or 0 and last_pk.

```
{
    "next": "/v1/instances/syncano/codeboxes/1/schedules/4/traces/?direction=1&last_pk=11268",
    "prev": "/v1/instances/syncano/codeboxes/1/schedules/4/traces/?direction=0&last_pk=11367",
    "objects": [
        {
            "id": 11367,
            "status": "success",
            "executed_at": "2015-01-21T12:18:08.791108Z",
            "duration": 365,
            "result": "uh ah!",
            "links": {
                "self": "/v1/instances/syncano/codeboxes/1/schedules/4/traces/11367/"
            }
        },
        ...
}
```

#### Changing page size

Number of items by page is set with `page_size` query parameter.

#### Different orderings

On data object endpoint, we can order by different values with `order_by` parameter.

### Some ideas behind the api design

The syncano API is a nested api, most of the endpoints are prefixed by instances/<instance_name>/something/

This is because of tenancy architecture of syncano backend. Every instance is completely separated from another, so class with name 'testclass' on one instance can be a totally different class on different instances with the same name there.


### HATEOAS - links

We use HATEOAS in our rest api. If you don't know what a HATEOAS is, [grab a link](http://stackoverflow.com/questions/9192648/hateoas-concise-description).

In syncano API, every resource has links to other resources. Below is the example for the instance:

```
{
    "name": "mytestinstance",
    "description": "this and that",
    "owner": {
        "id": 10,
        "email": "justyna.ilczuk@gmail.com",
        "first_name": "Justyna",
        "last_name": "Ilczuk"
    },
    "created_at": "2015-01-21T15:25:31.887657Z",
    "updated_at": "2015-01-21T15:25:31.909999Z",
    "role": "full",
    "links": {
        "triggers": "/v1/instances/mytestinstance/triggers/",
        "self": "/v1/instances/mytestinstance/",
        "invitations": "/v1/instances/mytestinstance/invitations/",
        "admins": "/v1/instances/mytestinstance/admins/",
        "classes": "/v1/instances/mytestinstance/classes/",
        "codebox_runtimes": "/v1/instances/mytestinstance/codeboxes/runtimes/",
        "webhooks": "/v1/instances/mytestinstance/webhooks/",
        "api_keys": "/v1/instances/mytestinstance/api_keys/",
        "codeboxes": "/v1/instances/mytestinstance/codeboxes/"
    }
}
```
### Options method on endpoints

You might find it useful to make OPTIONS requests on endpoints to get more details about them.

```
HTTP 200 OK
Content-Type: application/json
Vary: Accept
Allow: GET, POST, HEAD, OPTIONS

{
    "name": "Instance List",
    "description": "",
    "renders": [
        "application/json",
        "text/html"
    ],
    "parses": [
        "application/json",
        "application/x-www-form-urlencoded",
        "multipart/form-data"
    ],
    "actions": {
        "POST": {
            "name": {
                "type": "string",
                "required": true,
                "read_only": false,
                "label": "name",
                "max_length": 64
            },
            "description": {
                "type": "string",
                "required": false,
                "read_only": false,
                "label": "description"
            },
            "owner": {
                "id": {
                    "type": "integer",
                    "required": false,
                    "read_only": true,
                    "label": "ID"
                },
                "email": {
                    "type": "email",
                    "required": true,
                    "read_only": false,
                    "label": "email address",
                    "max_length": 254
                },
                "first_name": {
                    "type": "string",
                    "required": false,
                    "read_only": false,
                    "label": "first name",
                    "max_length": 35
                },
                "last_name": {
                    "type": "string",
                    "required": false,
                    "read_only": false,
                    "label": "last name",
                    "max_length": 35
                }
            },
            "created_at": {
                "type": "datetime",
                "required": false,
                "read_only": true,
                "label": "created at"
            },
            "updated_at": {
                "type": "datetime",
                "required": false,
                "read_only": true,
                "label": "updated at"
            },
            "role": {
                "type": "field",
                "required": false,
                "read_only": true
            },
            "links": {
                "type": "field",
                "required": false,
                "read_only": true
            }
        }
    }
}
```


### Schema endpoint

API has a schema endpoint that describes every endpoint, methods and parameters in a consistent way.

https://syncanotest1-env.elasticbeanstalk.com/v1/schema/

[Creating classes](http://syncano.readme.io/v4.0/docs/classes-1).
[Data objects management](http://syncano.readme.io/v4.0/docs/data-object-management),

