---
layout: post
title:  "SQLAlchemy models with JSON fields"
date:   2014-04-24
categories: sqlalchemy python
---
The purpose of this article was to show how to show how to transparently
serialize dictionaries as JSON fields in your database through SQLAlchemy and
access them from your code as dictionaries.

Quite often the case is that some request is supposed to return a set of these
objects to user. The use case might be e.g. a REST interface and user is
querying, let's say, blog posts. Some of the post objects however might include
fields which are not meant for or are just not necessary for the user. Primary
keys for example. This is why I have adopted way of creating an super class
Serializable with a method to turn all the selected keys and their values in a
model into a dictionary.


{% highlight python %}
class Serializable(object):
    __public__ = []

    def to_dict(self):
        d = {}
        for field in self.__public__:
            value = getattr(self, field)
            if value:
                d[field] = value
        return d
{% endhighlight %}

To use this in models, the model must inherit from Serializable. In case of JSON
fields the thing is a bit problematic. In this specific case the JSON field is
only to used by clients, so no efficient queries are required for the
information in the field. However dictionaries and JSON not being common data
types in SQL databases the model must include a special handling to serialize
and deserialize the dictionary into JSON string presentation and back. This is a
quite useful case for Python descriptors. Especially in SQLAlchemy there is a
special kind of descriptor like entities known as [hybrid properties][hybrid]

{% highlight python %}
class Example(Base, Serializable):
    __tablename__ = 'example'
    __public__ = ['column1', 'column_json']

    id = Column(Integer, primary_key=True)
    column1 = Column(String(32))
    _column_json = Column("column_json", Text)

    @hybrid_property
    def column_json(self):
        try:
            value = json.loads(self._column_json)
        except:
            value = {}
        return value
{% endhighlight %}

After this it's quite convenient to use your model

{% highlight python %}
>>> example = Example()
>>> example.column1 = "String content"
>>> example.column_json = {'foo': 42}
>>> session.add(example)
>>> session.commit()

>>> obj = session.query(Example).first()
>>> print obj.column1
String content
>>> print obj.column_json
{u'foo': 42}
{% endhighlight %}

Link to full [source code][source]

[hybrid]:   http://docs.sqlalchemy.org/en/rel_0_7/orm/extensions/hybrid.html
[source]:   http://hhamalai.net/media/code/sqlalchemy-json-fields.py
