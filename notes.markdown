---
layout: page
title: Quick notes
permalink: /notes/
---

On OS X watch directory (./src/) for changes and run executable on any change. #inotify

{% highlight bash %}
fswatch -o ./src/ | xargs -n1 /path/to/custom-build-script.sh
{% endhighlight %}



