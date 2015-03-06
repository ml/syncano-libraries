
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

[General information](http://syncano.readme.io/v4.0/docs/general-information).
[Creating classes](http://syncano.readme.io/v4.0/docs/classes-1).
[Data objects management](http://syncano.readme.io/v4.0/docs/data-object-management),

