## Writing a mobile library for syncano v4 (android / iOS)

What are the most important parts of syncano? Definitely classes and objects. Apart from that, there are other things such as codeboxes with webhooks, triggers and schedules that encapsulate custom logic executed on the server side.

From the mobile library perspective, creating new classes programmatically  doesn't make a lot of sense. Data would be probably modeled from CLI or web GUI. Also creating and configuring codeboxes shouldn't be a major focus of mobile library.

## DataObjects

Mobile library should focus on a great experience with using data objects of different classes, on retrieving, creating, updating, deleting and reacting to changes (real-time support is still under development). 

So, take a look at

` /v1/instances/:instance_name/classses/`

and

`/v1/instances/:instance_name/classses/:class_name/objects/`

## CodeBoxes

Focus on running codeboxes, for example with already available webhook.

## What to ignore

The helpful picture using our swager docs.
![](http://i.imgur.com/jSKPmgl.png)

### /account/
For the alpha version we don't need creating new admins, accepting invitations or logging with password.

### /billing/

Firstly, billing isn't ready yet. Secondly, this part of api is used for charging syncano admins and checking account balances.

### /metrics/

Unless you're building a mobile dashboard for syncano, you probably don't need to use this part of the api. With this part of the api, you can learn how many api calls were made to your instance with given apikey during some time interval, etc.
