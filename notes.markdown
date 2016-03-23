---
layout: page
title: Quick notes
permalink: /notes/
---

### Vim must have plugins
- [NERD Tree](https://github.com/scrooloose/nerdtree)
- [Syntastic](https://github.com/scrooloose/syntastic)
- [vim-airline](https://github.com/vim-airline/vim-airline)


### On OS X watch directory (./src/) for changes and run executable on any change (like inotify)

{% highlight bash %}
fswatch -o ./src/ | xargs -n1 /path/to/custom-build-script.sh
{% endhighlight %}

### User input sanitation and validation on with Node
- [express-validator](https://github.com/ctavan/express-validator)
- [joi](https://github.com/hapijs/joi)

