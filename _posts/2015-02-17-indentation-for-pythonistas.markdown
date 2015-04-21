---
layout: post
title:  "Indentation for Pythonistas"
date:   2015-01-27 09:53:42
categories: indentation python
---

I enjoy reading elegantly and properly written Python applications. Readability
counts. To enchance readability [PEP 8][pep8] provides a list of coding
conventions to follow when writing code that is supposed to be easily readable
by public. 

PEP 8 is a must read; setup your editor run automated PEP 8 checks every once in
a while. For vim there is [Syntastic][syntastic] to carry out that task.

Only thing I'm not a huge fan in PEP 8 is the 79 character limit for the length
of a line. As PEP 8 itself states, lines with less than one hundred characters
should not be a major concern, when used consistently. But when a line must
be splitted with a line break, the correct indentation can make it still a
bit more readable. Rudimentary indenation rule is to use four spaces as an
unit for indentation. Some further examples follows here.

For a list of arguments the PEP 8 checkers tend to complain if the indentation does not match the
preceding lines. Naive solution is to indent the subsequent lines to have the
same level of indentation as the first argument of the list on the first line.

{% highlight python %}
# Lines too long
info['hours'] = usage_controller._hours_for(instance,
                                            timeutils.datetime.datetime(1970, 1, 1),
                                            timeutils.datetime.datetime.now())
{% endhighlight %}

The outcome is not that fancy. Subsequent lines are likely to encounter more
E501 errors from the PEP 8 checker (E501: line length over 79 characters).

To get rid of the ridiculously wide indenations "hanging indenation" can be used. 
With this all lines are indented after opening parenthesis by one four space block.

{% highlight python %}
info['hours'] = usage_controller._hours_for(
   instance,
   timeutils.datetime.datetime(1970, 1, 1), 
   timeutils.datetime.datetime.now())
{% endhighlight %}

When a nested structure needs to be indented, multiple levels of indentation are required.

{% highlight python %}
req = urllib2.Request(
    '%s/%s/flavors/detail' % (
        self.nova_endpoint_url,
        self.tenant_id
    ), None, headers)
{% endhighlight %}

I tend to prefer above over strictly using hanging indentation everywhere, as the style above feels less extensive and taunting.

{% highlight python %}
req = urllib2.Request(
    '%s/%s/flavors/detail' % (
        self.nova_endpoint_url,
        self.tenant_id
    ),
    None,
    headers)
{% endhighlight %}

Another indentation style allowed by pep8 checker without hanging indentation, but probably should not be used as indentation is lost.

{% highlight python %}
req = urllib2.Request('%s/%s/flavors/detail' % (
    self.nova_endpoint_url,
    self.tenant_id
), None, headers)
{% endhighlight %}

Overly long import statements, like the one below, can be indented by two ways.
{% highlight python %}
from nova.api.openstack.compute.contrib.simple_tenant_usage import SimpleTenantUsageController
{% endhighlight %}

My personal favourite is to to use parenthesis with hanging indentation.

{% highlight python %}
# Using parenthesis
from nova.api.openstack.compute.contrib.simple_tenant_usage import (
    SimpleTenantUsageController)
{% endhighlight %}

But the backslash can also be used.

{% highlight python %}
# Using line continuation
from nova.api.openstack.compute.contrib.simple_tenant_usage import \
    SimpleTenantUsageController
{% endhighlight %}


Consistency promotes readability and elegancy, so try pay attention to
be consistent with your style.

[pep8]:         http://legacy.python.org/dev/peps/pep-0008/
[syntastic]:    https://github.com/scrooloose/syntastic
