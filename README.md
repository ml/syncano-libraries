
# Syncano library developer starter

### !!! This guide refers to the early alpha version of syncano v4. !!!

[Python Examples](https://github.com/Syncano/syncano-python/tree/release/4.0)

### First interaction with the API
Current staging infrastructure is located at:
https://syncanotest1-env.elasticbeanstalk.com/
But it also has an dns alias to:
https://v4.hydraengine.com/

From there main page, you can navigate to:

- docs [/docs/](https://syncanotest1-env.elasticbeanstalk.com/docs/) - swagger docs
- api [/v1/](https://syncanotest1-env.elasticbeanstalk.com/v1/) - where is the first version of the api

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

### How authorization works?

There are few options there:

- add authorization header
```
authorization_header = 'token %s' % api_key
HTTP_AUTHORIZATION=authorization_header
```
- add apikey header
```
HTTP_X_API_KEY=api_key
```
- add query parameter to request
```
https://syncanotest1-env.elasticbeanstalk.com/v1/instances/?api_key=my-very-secret-key
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

## Creating new classes and objects

To create objects, you first need to create a class.

A data to create a class might look like below

```
data = {
    'description': 'test2',
    'name': 'test2',
    'icon': 'ikona',
    'color': '#123123',
    'schema': [{'name': 'a', 'type': 'string'}]
}
```
Schema is a list of dictionaries describing fields:
```
schema = [{"name": "my_value", "type": "string"}, {"name": "second_value", "type": "float"}]
```
Instance name is validated by regex:
```
'^[A-Za-z][A-Za-z0-9\-_]*$'
```

### Types of fields
They following types of fields are supported now:

- 'string' - CharField - max_length: 128
- 'text': TextField - max_length:  32000
- 'integer':  IncrementableIntegerField
- 'float':  IncrementableFloatField
- 'boolean': NullBooleanField
- 'datetime': DateTimeField
- 'file': TextField
- 'reference': ReferenceField

Required values for each field dictionary:
required_keys = `{'name', 'type'}`

### Indexes
You can add indexes to most of the file types. Types without indexes are

- text
- file


For the rest, you can add `order_index` and `filter_index`. Fields with `order_index` can be used in `order_by` query, and fields with `filter_index` can be used in filtering in objects endpoint.

Example of class' schema with indexes
```
 {
            'name': 'test', 'icon': 'test', 'color': '#ffffff', 'description': 'test test',
            'schema': [
                {'name': 'string', 'type': 'string', 'order_index': True, 'filter_index': True},
                {'name': 'text', 'type': 'text'},
                {'name': 'integer', 'type': 'integer', 'order_index': True, 'filter_index': True},
                {'name': 'float', 'type': 'float', 'order_index': True, 'filter_index': True},
                {'name': 'bool', 'type': 'boolean', 'order_index': True, 'filter_index': True},
                {'name': 'datetime', 'type': 'datetime', 'order_index': True, 'filter_index': True},
                {'name': 'file', 'type': 'file'},
                {'name': 'ref', 'type': 'reference', 'order_index': True, 'filter_index': True, 'target': 'self'}
]}

```

### Reference field

If a field in schema is of type `reference`, there has to be a specified `target`. Target can be either `self` or name of some other class.

## Creating objects

To create objects of given class, navigate to `objects` link of the created class. There will be endpoint with proper validation. This endpoint should look as follows:

`/v1/my_instance/classes/my_class/objects`
