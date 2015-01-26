# High level libraries spec

## Connection object to syncano and easy authorization

```
import syncano

connection = syncano.connect(email='', password='')
```

## Wrappers around

- instances

###Data:

- classes (defining schemas, data modelling)
- data_objects (preferably with native classes suport, field validation and so on)

###CodeBoxes:

- codeboxes
- webhooks
- codebox schedules (normal and periodic)
- codebox traces (results of the codebox execution)
- triggers - codeboxes that will run when something happens in an application

###Account:

- metrics
- billing
- invitations
- managing apikeys
- managing permissions (incomplete)
- registration
- auth

###Users:
Will come later.

###Sync:
That will come later - but it's important now how it should look like.

##Other features:

- easy iteration over result sets from api (seemles paging)
- use hyperlinks provided from API
- use info from api to develop some additional validation

# Ideas

```
import syncano

connection = syncano.connect(email='', password='')
Instance = connection.models.Instance
```

# Create
```
i = Instance()
i.name = 'test'
i.save()
```
or 
```
Instance.create(name='test')  # will validated automatically and saved
```


# Query (this will change)
```
Instance.list(limit=10)
Instance.detail(name='test')
Instance.delete(name='test')
Instance.update(name='test', {'name': 'not test'})
```

## Interactions with `classes` and `data_objects`

I was thinking about a few approaches there:

### Relating to classes via instance directly

```
score_class = my_instance.classes.create(name="score", schema=[{"name": "username", "type": "string"}, {"name": "result", "type": "integer"}])
scores = score_class.data_objects

alice_score = scores.new(name='Alice', result=5)
bob_score = scores.new(name='Bob')
bob_score.result = 1337
bob_score.save()
```

Another idea:

### Setting a default instance for convenience
```
connection.set_default_instance(name="my_instance")
# or
connection.set_default_instance(my_instance)
```


## Creating classes

```
Klass = connection.models.Klass

score_class = Klass(name="score", schema=[{"name": "username", "type": "string"}]).register(override=True)

or

score_class = Klass.get(name="score")

# data_objects before `new`
alice_score = score_class.data_objects.new(name="Alice", score=123)
alice_score.save()

or

class Score(SyncanoClass):
	override = True  # override if schema differs?
	name = StringField(blank=False)
	result = IntegerField()

alice_score = Score(name="Alice", result=234)
alice_score.result = 2
alice_score.save()
alice_score.delete()  # etc.

```

## Accessing `classes` and `data_objects`

### DataObject with specified class
```
connection.set_default_instance(my_instance)
DataObject = connection.models.DataObject
bob_score = DataObject(klass="score")
bob_score.result = 1234
bob_score.name = "Bob"
bob_score.save()

eve_score = DataObject.create(klass="score", result=1234, name="Eve")

score_klass = my_instance.classes.Score 
or
score_klass = connection.models.Klass.get(name="score")

print score_klass.name
print score_klass.schema
print score_klass.data_objects

bob_score = score_klass.data_objects.new(name="Bob")
bob_score.result = 123
bob_score.save()

alice_score = DataObject(klass=score_klass, result=123, name="Alice")
alice_score.save()
```

### Pseudo native class
```
Score = connection.models.Klass.get(name="score") 
alice_score = Score(name="Alice", result=12345)
alice_score.save()
```

## Ideas about synchronization

Callbacks or (and) promises, depending on the language.

## CodeBox ideas

### Regular codeboxes
```
CodeBox = connection.models.CodeBox

# create codebox with soure code from the file, python runtime and additional config
with open("codebox_hello.py") as f:
	my_codebox = CodeBox(source=f.read(), runtime_name="python", config={"facebook_key": "123"})
	my_codebox.save()

# intermeditate execution
result = my_codebox.run({"something": "123"})

# if sync
print result

# if async
print result.wait()

# or
print result.wait(timeout=3)

```


### Creating periodic schedules
```
CodeBoxSchedule = connection.models.CodeBoxSchedule

schedule = CodeBoxSchedule(codebox=my_codebox, interval_sec==300).save()

# after some time

for trace in schedule.traces.list():
	print trace.status, trace.duration, trace.result

# it's not possible now, but it will be nice to have an easy way of disabling periodic schedules without removing traces

schedule.disable()
schedule.save()
```

### Webhooks

```
Webhook = connection.models.Webhook

hello_webhook = Webhook(codebox=my_codebox, slug="hello")
hello_webhook.save()
result = hello_webhook.run({"name": "Alice"})
```

### Triggers

```
Trigger = connection.models.Trigger

mail_after_new_highest_score = Trigger(codebox=my_mailing_codebox, class=score_klass).save()

score_klass.data_objects.create(result=9000, name="Lucy")

# new mail in the inbox!
```

Proceeding examples aren't final, the discussion is still open.
